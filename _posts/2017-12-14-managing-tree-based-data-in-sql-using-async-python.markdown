---
layout: post
title:  "Managing tree-based data in SQL using async Python"
date:   2017-12-14 12:20:45 +0500
categories: python asyncio postgresql
---
# This article details how async Python can be used to develop a simple API for working with tree-based data.

## Intro

Tree-based data has often been used to represent the parent-child relationships in the user
business processes. There are several well-known models for storing such hierarchical data,
for example [adjacency list](https://en.wikipedia.org/wiki/Adjacency_list),
[nested set](https://en.wikipedia.org/wiki/Nested_set_model),
[closure table](https://coderwall.com/p/lixing/closure-tables-for-browsing-trees-in-sql), etc.
We do not discuss which the best one is, but trying to show how one of those models can be
implemented using Python.

## Objectives

Let's develop a simple system that allows user to leave comments. Now it doesn't matter what
exactly the user can comment. The API being developed must satisfy the following most important
requirements.

* The user has to be authorized in order to make other API calls.
* There is a method to leave a new comment.
* There is a method to search comments by content.
* There is a method to retrieve a sub-tree of comment.
* All the API calls speak JSON.

Let's define some technical properties of our system. We would use

* async Python as a backend to handle many concurrent users,
* PostgreSQL as a RDBMS to exploit its full text search,
* SQLAlchemy to construct SQL queries,
* Swagger UI as a demonstration GUI for our API.

## Closure table

We would use a combination of closure table and adjacency list models. Closure table model
usually requires two tables to declare.

{% highlight python %}
import sqlalchemy as sa
 
meta = sa.MetaData()
	 
comments = sa.Table(
    'comments_comments', meta,
    sa.Column('id', sa.Integer, primary_key=True),
    sa.Column('content', sa.String, nullable=False)
)
 
comments_tree = sa.Table(
    'comments_tree', meta,
    sa.Column('id', sa.Integer, primary_key=True),
    sa.Column('ancestor_id', sa.Integer, sa.ForeignKey(comments.c.id), nullable=False),
    sa.Column('nearest_ancestor_id', sa.Integer, sa.ForeignKey(comments.c.id), nullable=False),
    sa.Column('descendant_id', sa.Integer, sa.ForeignKey(comments.c.id), nullable=False),
    sa.Column('depth', sa.Integer, nullable=False)
)
{% endhighlight %}

So far we've declared the **comments** table to store user comments, and the **comments_tree**
table to store parent-child comment relationships. Each row in the second one is a relation
between two comments. The columns have the following meaning:

* _ancestor_id_ is an id of the parent comment. It might be not a direct parent, it may be a grand parent or a grand-grand parent for instance. Each comment is an ancestor of itself at the same depth.
* _descendant_id_ is an id of a child comment.
* _depth_ is a level describing how deep in the tree the comment is. All the root comments have zero depth.

However, this closure table contains only a plain info about data relations and doesn't allow
to restore the ordered tree structure. To make it possible we'd like to add an additional column.

* _nearest_ancestor_id_ is an id of the direct parent of a comment.

Now it's possible to reproduce an ordered tree structure using info about a relation between a
comment and its direct parent.

Let's see how to fill out these tables by example. To create the first comment we're just putting
a text into _comments_ table.

<style type="text/css">
.comments-structure-tbl  {border-collapse:collapse;border-spacing:0;margin:0px auto;}
.comments-tbl  {border-collapse:collapse;border-spacing:0;border-color:#aaa;margin:0px auto;}
.comments-tbl td{font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:1px;overflow:hidden;word-break:normal;border-color:#aaa;color:#333;background-color:#fff;}
.comments-tbl th{font-family:Arial, sans-serif;font-size:14px;font-weight:normal;padding:10px 5px;border-style:solid;border-width:1px;overflow:hidden;word-break:normal;border-color:#aaa;color:#fff;background-color:#f38630;}
.comments-tbl .tg-first{vertical-align:top}
.comments-tree-tbl  {border-collapse:collapse;border-spacing:0;border-color:#999;border:none;margin:0px auto;}
.comments-tree-tbl td{font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-color:#999;color:#444;background-color:#F7FDFA;}
.comments-tree-tbl th{font-family:Arial, sans-serif;font-size:14px;font-weight:normal;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-color:#999;color:#fff;background-color:#26ADE4;}
.comments-tree-tbl .tg-second{background-color:#D2E4FC;text-align:right;vertical-align:top}
.comments-tree-tbl .tg-first{text-align:right;vertical-align:top}
.comments-tree-tbl .tg-header{vertical-align:top}
</style>

<table class="comments-tbl">
<tbody>
	<tr>
		<th class="tg-first">id</th>
		<th class="tg-first">content</th>
	</tr>
	<tr>
	    <td class="tg-first">1</td>
	    <td class="tg-first">Can anybody explain me the difference between Bitcoin and Etherium?</td>
	</tr>
</tbody>
</table>

We would also mark that comment as a parent (or ancestor) of itself.

<table class="comments-tree-tbl">
<tbody>
	<tr>
		<th class="tg-header">id</th>
		<th class="tg-header">ancestor_id</th>
		<th class="tg-header">nearest_ancestor_id</th>
		<th class="tg-header">descendant_id</th>
		<th class="tg-header">depth</th>
	</tr>
	<tr>
	    <td class="tg-second">1</td>
		<td class="tg-second">1</td>
		<td class="tg-second">1</td>
		<td class="tg-second">1</td>
		<td class="tg-second">0</td>
	</tr>
</tbody></table>

To create a child comment we additionally put a relation with its grand parent (a non direct
ancestor with non-zero depth). The tables are changed as shown below.

<table class="comments-tbl">
<tbody>
	<tr>
		<th class="tg-first">id</th>
		<th class="tg-first">content</th>
	</tr>
	<tr>
		<td class="tg-first">1</td>
		<td class="tg-first">Can anybody explain me the difference between Bitcoin and Etherium?</td>
	</tr>
	<tr>
		<td class="tg-first">2</td>
		<td class="tg-first">I'm sure no one can.</td>
	</tr>
</tbody>
</table>

<table class="comments-tree-tbl">
<tbody>
	<tr>
		<th class="tg-header">id</th>
		<th class="tg-header">ancestor_id</th>
		<th class="tg-header">nearest_ancestor_id</th>
		<th class="tg-header">descendant_id</th>
		<th class="tg-header">depth</th>
	</tr>
	<tr>
		<td class="tg-second">1</td>
		<td class="tg-second">1</td>
		<td class="tg-second">1</td>
		<td class="tg-second">1</td>
		<td class="tg-second">0</td>
	</tr>
	<tr>
		<td class="tg-first">2</td>
		<td class="tg-first">2</td>
		<td class="tg-first">2</td>
		<td class="tg-first">2</td>
		<td class="tg-first">1</td>
	</tr>
	<tr>
		<td class="tg-second">3</td>
		<td class="tg-second">1</td>
		<td class="tg-second">1</td>
		<td class="tg-second">2</td>
		<td class="tg-second">1</td>
	</tr>
</tbody></table>

Now we can see that the number of the relations and the depth are growing together. That's why
closure table model has often been criticized. Obviously, in order to add an element with high
depth value to the tree, we'll have to store lots of relations of the element and all its
ancestors.

But it's not as bad as it seems. In a real case adding an element with a depth of 200 or 1000 may
not affect the performance of a server in serious way. In fact, such a situation probably will
never happen (except we're commenting the news about our national team losing a soccer match).
In many cases, the time of data selection is more significant than the insertion time.

Back to our example. The full comment branch is looking like that.

<table class="comments-structure-tbl">
	<tbody>
		<tr>
			<td>1</td>
			<td>&nbsp; Can anybody explain me the difference between Bitcoin and Etherium?</td>
		</tr>
		<tr>
			<td>2</td>
		    <td>&nbsp;&nbsp;&nbsp;&nbsp; I'm sure no one can.</td>
		</tr>
		<tr>
			<td>6</td>
			<td>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Why are you so sure?</td>
		</tr>
		<tr>
		    <td>7</td>
		    <td>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; This is a blog about cooking!</td>
		</tr>
		<tr>
		    <td>3</td>
		    <td>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; The same request.</td>
		</tr>
		<tr>
		    <td>4</td>
		    <td>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; No problem, I'll give an explanation paper later.</td>
		</tr>
		<tr>
		    <td>5</td>
		    <td>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Send me it by email then.</td>
		</tr>
	</tbody>
</table>

Let's see how this branch will be represented in the tables.

<table class="comments-tbl">
<tbody>
<tr>
    <th class="tg-first">id</th>
    <th class="tg-first">content</th>
</tr>
<tr>
    <td class="tg-first">1</td>
    <td class="tg-first">Can anybody explain me the difference between Bitcoin and Etherium?</td>
</tr>
<tr>
    <td class="tg-first">2</td>
    <td class="tg-first">I'm sure no one can.</td>
</tr>
<tr>
    <td class="tg-first">3</td>
    <td class="tg-first">The same request.</td>
</tr>
<tr>
    <td class="tg-first">4</td>
    <td class="tg-first">No problem, I'll give an explanation paper later.</td>
</tr>
<tr>
    <td class="tg-first">5</td>
    <td class="tg-first">Send me it by email then.</td>
</tr>
<tr>
    <td class="tg-first">6</td>
    <td class="tg-first">Why are you so sure?</td>
</tr>
<tr>
    <td class="tg-first">7</td>
    <td class="tg-first">This is a blog about cooking!</td>
</tr>
</tbody>
</table>

<table class="comments-tree-tbl">
<tbody>
<tr>
    <th class="tg-header">id</th>
    <th class="tg-header">ancestor_id</th>
    <th class="tg-header">nearest_ancestor_id</th>
    <th class="tg-header">descendant_id</th>
    <th class="tg-header">depth</th>
</tr>
<tr>
    <td class="tg-second">1</td>
    <td class="tg-second">1</td>
    <td class="tg-second">1</td>
    <td class="tg-second">1</td>
    <td class="tg-second">0</td>
</tr>
<tr>
    <td class="tg-first">2</td>
    <td class="tg-first">2</td>
    <td class="tg-first">2</td>
    <td class="tg-first">2</td>
    <td class="tg-first">1</td>
</tr>
<tr>
    <td class="tg-second">3</td>
    <td class="tg-second">1</td>
    <td class="tg-second">1</td>
    <td class="tg-second">2</td>
    <td class="tg-second">1</td>
</tr>
<tr>
    <td class="tg-first">4</td>
    <td class="tg-first">3</td>
    <td class="tg-first">3</td>
    <td class="tg-first">3</td>
    <td class="tg-first">1</td>
</tr>
<tr>
    <td class="tg-second">5</td>
    <td class="tg-second">1</td>
    <td class="tg-second">1</td>
    <td class="tg-second">3</td>
    <td class="tg-second">1</td>
</tr>
<tr>
    <td class="tg-first">6</td>
    <td class="tg-first">4</td>
    <td class="tg-first">4</td>
    <td class="tg-first">4</td>
    <td class="tg-first">1</td>
</tr>
<tr>
    <td class="tg-second">7</td>
    <td class="tg-second">1</td>
    <td class="tg-second">1</td>
    <td class="tg-second">4</td>
    <td class="tg-second">1</td>
</tr>
<tr>
    <td class="tg-first">8</td>
    <td class="tg-first">5</td>
    <td class="tg-first">5</td>
    <td class="tg-first">5</td>
    <td class="tg-first">2</td>
</tr>
<tr>
    <td class="tg-second">9</td>
    <td class="tg-second">4</td>
    <td class="tg-second">4</td>
    <td class="tg-second">5</td>
    <td class="tg-second">2</td>
</tr>
<tr>
    <td class="tg-first">10</td>
    <td class="tg-first">1</td>
    <td class="tg-first">4</td>
    <td class="tg-first">5</td>
    <td class="tg-first">2</td>
</tr>
<tr>
    <td class="tg-second">11</td>
    <td class="tg-second">6</td>
    <td class="tg-second">6</td>
    <td class="tg-second">6</td>
    <td class="tg-second">2</td>
</tr>
<tr>
    <td class="tg-first">12</td>
    <td class="tg-first">2</td>
    <td class="tg-first">2</td>
    <td class="tg-first">6</td>
    <td class="tg-first">2</td>
</tr>
<tr>
    <td class="tg-second">13</td>
    <td class="tg-second">1</td>
    <td class="tg-second">2</td>
    <td class="tg-second">6</td>
    <td class="tg-second">2</td>
</tr>
<tr>
    <td class="tg-first">14</td>
    <td class="tg-first">7</td>
    <td class="tg-first">7</td>
    <td class="tg-first">7</td>
    <td class="tg-first">3</td>
</tr>
<tr>
    <td class="tg-second">15</td>
    <td class="tg-second">6</td>
    <td class="tg-second">6</td>
    <td class="tg-second">7</td>
    <td class="tg-second">3</td>
</tr>
<tr>
    <td class="tg-first">16</td>
    <td class="tg-first">2</td>
    <td class="tg-first">6</td>
    <td class="tg-first">7</td>
    <td class="tg-first">3</td>
</tr>
<tr>
    <td class="tg-second">17</td>
    <td class="tg-second">1</td>
    <td class="tg-second">6</td>
    <td class="tg-second">7</td>
    <td class="tg-second">3</td>
</tr>
</tbody>
</table>

As it shown above, there is a root comment with zero depth within the branch and the max depth
(or level) of comments is 3. Now let's see how we can deal with these tables in Python using
SQLAlchemy.

## SQL queries

### Inserting a node to the tree



Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
