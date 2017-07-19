## Summary

Flask is the micro web framework written by Python.
It is micro, so, there are valious way to make constitusion.
I want to show a sample way with such as MVC pattern bellow.

Demo : http://172.104.82.128/
Github : https://github.com/bamboo-yujiro/flask-mvc-sample

## Constitution

```
├── app.py
├── bundle.py
├── env.py.sample
├── __init__.py
├── logger.py
├── main.py
├── manage.py
├── migrate.py
├── migrations
│   ├── alembic.ini
│   ├── README
│   ├── script.py.mako
│   └── versions
│       └── 99d75ed4b967_.py
├── myapp
│   ├── controllers
│   │   ├── general_base.py
│   │   ├── __init__.py
│   │   ├── memos.py
│   │   ├── top.py
│   │   └── users.py
│   ├── db
│   │   └── __init__.py
│   ├── __init__.py
│   ├── management
│   │   ├── commands
│   │   │   ├── command.py
│   │   │   ├── __init__.py
│   │   │   └── logs.py
│   │   └── __init__.py
│   ├── models
│   │   ├── common.py
│   │   ├── __init__.py
│   │   ├── memo.py
│   │   ├── memo_tag.py
│   │   ├── tag.py
│   │   └── user.py
│   └── valiables
│       ├── hoge.py
│       └── __init__.py
├── README.md
├── requirements.txt
├── static
│   ├── css
│   │   ├── reset.css
│   │   └── style.css
│   ├── dist
│   │   ├── ca812cc8cc6a5ff3e2430ebc9264eb6d.js
│   │   └── fc615c34010dbd7155980a403c0584db.js
│   ├── gen
│   │   └── packed.js
│   └── js
│       ├── jquery-1.12.2.min.js
│       └── jquery.cookie.js
├── templates
│   ├── layouts
│   │   └── default.html
│   ├── memos
│   │   ├── edit.html
│   │   ├── index.html
│   │   ├── new.html
│   │   ├── show.html
│   │   └── widgets
│   │       └── _back_list.html
│   ├── top
│   │   └── index.html
│   ├── users
│   │   ├── create.html
│   │   └── login.html
│   └── widgets
│       └── _pagination.html
└── uwsgi.ini.sample
```

### Description

main.py
script to run server

manage.py
for console

migrate.py
for migration of database

myapp/
main app directory. there are models, controllers, db in this.

static/
static folder for css, javascript, image

templates


## Model

To take an example of my demo app, relationship of database is like following.

===ER===

How implement this relationship?

### Memo <=> Tag （Many To Many）

``` myapp/models/memo.py

tmp_memo_tags = db.Table('memo_tags', ModelBase.metadata,
    db.Column('memo_id', db.Integer),
    db.Column('tag_id', db.Integer)
)

class Memo(ModelBase):
    __tablename__ = 'memos'
    ...
    # relationship (one to many) not necessarily
    memo_tags = db.relationship('MemoTag', backref='memo', lazy='dynamic', cascade="all", primaryjoin="Memo.id==MemoTag.memo_id", foreign_keys='MemoTag.memo_id')# one to many

    tags = db.relationship("Tag",
        backref="memos",
        primaryjoin="Memo.id==MemoTag.memo_id",
        secondaryjoin="MemoTag.tag_id==Tag.id",
        secondary=tmp_memo_tags,
        lazy='dynamic'
    )

```
#### Point
"Foreignkey" is the easist way when you want to add db relationship.

http://docs.sqlalchemy.org/en/latest/orm/basic_relationships.html#one-to-many

But I did'nt want to use "Foreignkey" because of making foreign key constraint on database , so, I implemented with next way.

It's way is using "primaryjoin" and "secondaryjoin" attribute.

```
primaryjoin="Memo.id==MemoTag.memo_id",
secondaryjoin="MemoTag.tag_id==Tag.id",
```
so, 
```
for tag in memo.tags.all():
    print(tag.name)  # => tag name will be printed!
```

If you don't want to make foreign key constraint, the example above is recommented.

If you implement it with "Foreignkey", when you migrate database with "Flask-Migrate",  database schema will be made with foreign key constraint.


Example of "One to many" or "One to One" is seen well,  but, I will introduce it for the moment. 

### One To Many
```
memo_tags = db.relationship('MemoTag', backref='memo', lazy='dynamic', cascade="all", primaryjoin="Memo.id==MemoTag.memo_id", foreign_keys='MemoTag.memo_id')# one to many
```

### One To One
```
user_password = db.relationship('UserPasword', primaryjoin="User.id==UserPassword.user_id", foreign_keys='UserPassword.user_id', uselist=False) # one to one
```
※ This example isn't related with this sample app.

