---
title: SQLAlchemy and Flask-admin
subtitle: 
date: '2016-08-08'
slug: sqlalchemy-flask-admin
---

## Using SQLAlchemy and Flask-admin

In my project, we have a data driven application, with a heavy focus on the
database portion. The application is such that the data required comes from
the government, so the database is populated with information from the
government and needs to be routinely updated. Apart from just cron jobs to
populate the database, there is also functionality to insert into certain
tables from a Flask application. The architecture of the application consists
of a PostgreSQL database and a Python program consisted of parsing scripts
that handle the ingestion of the data from the government. This forms the
backend, which has python workflows that create scheduled reports from the
database.

Flask admin comes into play here for certain tables because we have customer
information also within the same database, which sometimes needs to be
updated. Using the SQLAlchemy automap feature, we are able to create models of
certain tables from the database, and then we can use Flask-admin to create a
web interface that allows the admin to update customers. Automap base handles
the creation of a Python object that properly does the foreign key
relationships of every table. This means that I can create an object that has
a foreign key linkage, and when I commit that object into the PostgreSQL
database, the ORM inserts into all the required tables for you.

```
    from sqlalchemy import create_engine, MetaData
    from sqlalchemy.ext.automap import automap_base
    from app import app
    
    DBNAME_engine = create_engine(app.config['DB_URI'], echo=True)
    
    metadata = MetaData()
    CHOSEN_TABLES = [
                'customer',
                'customer_meter_detail',
                'registry',
                'address',
                'contact',
                'customer_bank_detail',
    ]
    metadata.reflect(DBNAME_engine, only=CHOSEN_TABLES)
    
    Base = automap_base(metadata=metadata)
    Base.prepare()
    
    Customer = Base.classes.customer
    Registry = Base.classes.registry
    CustomerMeter = Base.classes.customer_meter_detail
    Address = Base.classes.address
    Contact = Base.classes.contact
    CustomerBankDetail = Base.classes.customer_bank_detail
```

This differs from the usual usage of a web application where the database is
only ever handled as a single entity, so there is no need to hide some of the
tables from the rest of the domain. Since SQLAlchemy already has an object
created for each of the required tables, we can use these objects in Flask-
admin by extending the `ModelView` class.

```
    from flask.ext.admin.contrib.sqla import ModelView
    from flask.ext.admin import Admin
    
    admin = Admin(app, name='Editing Customer Info')
    admin.add_view(CustomerAdminView(Customer, session))
    admin.add_view(ContactAdminView(Contact, session))
    admin.add_view(AddressAdminView(Address, session))
```

Doing so, you now have an easy admin access for those selected tables.
