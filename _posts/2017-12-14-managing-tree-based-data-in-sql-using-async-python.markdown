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
<br />

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
</tbody>
</table>
<br />

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
<br />

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
</tbody>
</table>
<br />

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
<br />

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
<br />

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
<br />

As it shown above, there is a root comment with zero depth within the branch and the max depth
(or level) of comments is 3. Now let's see how we can deal with these tables in Python using
SQLAlchemy.

## SQL queries

### Inserting a node to the tree

Filling _comments_tree_ table can be done by using SQL
[INSERT INTO SELECT](https://www.w3schools.com/sql/sql_insert_into_select.asp) statement.
Using SQLAlchemy it can be written like shown below.

{% highlight python %}
import sqlalchemy as sa
 
ancestor = comments_tree.alias('ancestor')
descendant = comments_tree.alias('descendant')
nearest = comments_tree.alias('nearest')
 
sel = sa.select([
	descendant.c.ancestor_id,
	nearest.c.nearest_ancestor_id,
	ancestor.c.descendant_id,
	nearest.c.depth + 1
]).where(sa.and_(
	descendant.c.descendant_id == parent_id,
	ancestor.c.ancestor_id == comment_id,
	nearest.c.ancestor_id == parent_id,
	nearest.c.descendant_id == parent_id,
))

ins = comments_tree.insert().from_select([
    comments_tree.c.ancestor_id,
    comments_tree.c.nearest_ancestor_id,
    comments_tree.c.descendant_id,
    comments_tree.c.depth,
], sel)
{% endhighlight %}

That code produces the following SQL query.

{% highlight sql %}
INSERT INTO comments_tree
	(ancestor_id, nearest_ancestor_id, descendant_id, depth)
SELECT
	descendant.ancestor_id,
	nearest.nearest_ancestor_id,
	ancestor.descendant_id,
	nearest.depth + 1
FROM
	comments_tree AS descendant,
	comments_tree AS nearest,
	comments_tree AS ancestor
WHERE descendant.descendant_id = PARENT_ID
	AND ancestor.ancestor_id = COMMENT_ID
	AND nearest.ancestor_id = PARENT_ID
	AND nearest.descendant_id = PARENT_ID
{% endhighlight %}

### Selecting a sub-tree

Selecting the whole comment branch can be done like that.

{% highlight python %}
import sqlalchemy as sa
 
join = comments.join(
	comments_tree, comments.c.id == comments_tree.c.descendant_id
)

sel = sa.select([
	comments_tree.c.nearest_ancestor_id,
	comments.c.id,
	comments.c.content,
]).select_from(join).where(comments_tree.c.ancestor_id == comment_id)
{% endhighlight %}

The corresponding SQL query.

{% highlight sql %}
SELECT
	comments_tree.nearest_ancestor_id,
	comments_comments.id,
	comments_comments.content
FROM
    comments_comments
JOIN comments_tree ON comments_comments.id = comments_tree.descendant_id
WHERE comments_tree.ancestor_id = COMMENT_ID
{% endhighlight %}

### Deleting a sub-tree

To delete a comment and all its sub-comments we have to:

* figure out and remove all the relations in which the target comment is involved (meaning it's an ancestor or a nearest ancestor or a descendant),
* remove all the comments for which the target comment is an ancestor.

Using SQLAlchemy we can write it like that.

{% highlight python %}
import sqlalchemy as sa
 
remove = comments_tree.alias('remove')
descendant = comments_tree.alias('descendant')
 
sel = sa.select([remove.c.descendant_id, remove.c.id]).where(sa.and_(
	sa.or_(
		remove.c.ancestor_id == descendant.c.descendant_id,
		remove.c.nearest_ancestor_id == descendant.c.descendant_id,
		remove.c.descendant_id == descendant.c.descendant_id,
	),
	descendant.c.ancestor_id == comment_id,
))
{% endhighlight %}

SQL query.

{% highlight sql %}
SELECT
	remove.descendant_id,
    remove.id
FROM
	comments_tree AS remove,
	comments_tree AS descendant
WHERE (remove.ancestor_id = descendant.descendant_id
    OR remove.nearest_ancestor_id = descendant.descendant_id
	OR remove.descendant_id = descendant.descendant_id)
	AND descendant.ancestor_id = COMMENT_ID
{% endhighlight %}

That query will select all the ids of  the comments (the first column) and all the ids of the
comment relations (the second column) we need to delete.

## The details

### Schema changes

It's most likely that we'll want to show more info about comments, not only a text. For instance,
we may want to display a name of the author of the comment and the time when comment has been left.
It will cause the schema changes to be applied. We would probably want to use one of the database
schema management systems like
[SQLAlchemy Migrate](http://sqlalchemy-migrate.readthedocs.io/en/latest/index.html)
to simplify that.

### Executing SQL

Due to async Python we would use [aiopg](https://aiopg.readthedocs.io/en/stable/index.html) to
operate upon PostgreSQL. We can init and shutdown a connection pool as shown below.

{% highlight python %}
import aiopg.sa
from aiohttp import web
 
async def setup_pg(app):
	engine = await aiopg.sa.create_engine(DATABASE)
	app['db'] = engine

async def close_pg(app):
	app['db'].close()
	await app['db'].wait_closed()

app = web.Application()

app.on_startup.append(setup_pg)
app.on_cleanup.append(close_pg)
{% endhighlight %}

### Full-text search

PostgreSQL supports full-text search, we can exploit it using SQLAlchemy.

{% highlight python %}
import sqlalchemy as sa
from sqlalchemy.sql.expression import func
 
sel = sa.select([comments]).where(comments.c.content.match(
	sa.cast(func.plainto_tsquery('query text'), sa.TEXT)
))
{% endhighlight %}

That will produce the following SQL query.

{% highlight sql %}
SELECT
	comments_comments.id,
    comments_comments.content
FROM
	comments_comments
WHERE
	comments_comments.content @@ to_tsquery(
		CAST(plainto_tsquery('query text') AS TEXT)
	)
{% endhighlight %}

This is a basic full-text search usage. For more complex examples see the links below.

### User auth

To avoid unauthorized access to the API we would require an auth token being passed as a
parameter to each call. We can use [JSON Web Tokens](https://en.wikipedia.org/wiki/JSON_Web_Token)
to implement that. To figure out how to use JWT in Python, follow the links below.

### Swagger UI

To provide a demonstration GUI for the API we can use
[aiohttp-swagger](http://aiohttp-swagger.readthedocs.io/en/latest/).

{% highlight python %}
from aiohttp import web
from aiohttp_swagger import setup_swagger
 
app = web.Application()

setup_swagger(app, swagger_from_file='swagger.yaml')
{% endhighlight %}

# Conclusion

In this article we've developed a simple API that allows user to leave comments. We've learned
how to use a combination of closure table and adjacency list models to work with tree-based data.
The source code of the project is available in
[the repository](https://github.com/vyacheslav-bezborodov/closure-table) on GitHub.

# Links

* http://technobytz.com/closure_table_store_hierarchical_data.html
* https://www.postgresql.org/docs/9.5/static/textsearch.html
* https://steelkiwi.com/blog/jwt-authorization-python-part-1-practise/

