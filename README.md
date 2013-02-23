# A Rails3 application with MongoDB & MongoMapper on Windows environment

The purpose of this tutorial is to provide step-by-step instruction on how to build a Rails 3 Application using MongoDB and MongoMapper. This application is built on windows environment.

MongoDB is a NoSQL database that can be used as a replacement for SqlLite, MySQL, Postgresql or other sql relational databases.  NoSQL is a great alternative for web applications where scalability is required or data must be grouped together in greater numbers than the hard limits applied by sql-based databases.

MongoDB can be downloaded and installed from http://docs.mongodb.org/manual/contents/

Basic Ruby and Rails 3 knowledge will be required to follow this tutorial.

## Prerequisites:

You must have the following installed on your system.

*  Ruby 1.9.3
*  Rails 3.2.11
*  MongoDB 2.0.8

# Tutorial: Creating a MongoDB, MongoMapper Rails 3 Application

The following tutorial results in the sample application included in this repository.

###  Step 1: Build a rails app using the  --skip-active-record option

The very first thing we are going to do is build the Rails 3 Application framework. Since MongoDB isn't a SQL database, it does not use ActiveRecord to store data. We will instead be using the MongoMapper gem as a replacement for ActiveRecord.  Therefore, we will need to use '--skip-active-record' when generating our Rails 3 Application.

	$ rails new MongoTest --skip-active-record

Navigate to the application directory:

	$ cd MongoTest/

### Step 2: Modify the Gemfile

Now that we have generated our Rails 3 application framework, we should modify our Gemfile to include the necessary gems.

Include the mongo_mapper & bson_ext gem, after the rails gem:

bson_ext is the C Extensions for optimum MongoDB Ruby driver performance.

	gem 'mongo_mapper'
	gem "bson_ext"

### Step 3:  Run the bundle installer:

Now install the required gems, by typing at the command prompt:

	$ bundle install

If successful, you should see a list of gems and at the end of the list the declaration, "Your bundle is complete!"

### Step 4:  Create a MongoDB initializer

We now need to add a MongoDB initializer at 'config/initializers/mongo.rb'

Here we give the name of the database we want to use

Add the following code block:

	MongoMapper.connection = Mongo::Connection.new('localhost', 27017)
	MongoMapper.database = "#mongotest-#{Rails.env}"

	if defined?(PhusionPassenger)
   		PhusionPassenger.on_event(:starting_worker_process) do |forked|
     			MongoMapper.connection.connect if forked
   		end
	end

When our rails app starts, the above code makes a connection to the Mongo database, sets the database name and makes the database connection accessible to PhusionPassenger, if applicable.


### Step 5: Create a MongoDB Rake Task

We also need to add a MongoDB Rake task to tell rake that it doesn't need to do anything when we run "rake db:test:prepare".

	lib/tasks/mongo.rake

Add the following code:

	namespace :db do
  		namespace :test do
    			task :prepare do
      				# Stub out for MongoDB
    			end
  		end
	end

Save and exit mongo.rake.

### Step 6: Create a resource through scafolding

rails g scaffold expense expense_name:string date_spent:string amount:integer --skip-migration --orm  mongo_mapper

You can see in the expense model, instead of ActiveRecord, MongoMapper is used which behaves pretty much the same.

### Step 7: Run the server

In Rails 3, a rails server is started by typing:

	$ rails S

If all the requirements and gems are successfully installed, you should see the following:

	=> Booting WEBrick
	=> Rails 3.1.1 application starting in development on http://0.0.0.0:3000
	=> Call with -d to detach
	=> Ctrl-C to shutdown server
	[2012-01-12 17:31:06] INFO  WEBrick 1.3.1
	[2012-01-12 17:31:06] INFO  ruby 1.9.2 (2011-07-09) [x86_64-darwin11.1.0]
	[2012-01-12 17:31:06] INFO  WEBrick::HTTPServer#start: pid=34046 port=3000

If you see this, open a browser and navigate to: http://localhost:3000/expenses

You should see the listing page of the expense resource

Make sure the mongodb server is running

To kill the server, type CONTROL-c at the command prompt.

## Modeling with MongoDB and MongoMapper in Rails 3

### No Migration

There are no migrations in a MongoDB, MongoMapper Rails 3 Application. There is no ActiveRecord, database migrations or ActiveModel validations. The application data schema is structured and vailidated by MongoMapper in the model.

### Create a Model without ActiveRecord::Base

The model should not include "< ActiveRecord::Base" in the model's class declartion. For example, let's say we were creating a model called Movie, at app/models/movie.rb. The top line of the model would simply be:

	class Movies

### Include MongoMapper:Document in the Model

We have included the MongoMapper gem in the application through our Gemfile, but we must also include it in our models. Therefore, after our class declaration we must add the following line to our model:

	include MongoMapper::Document

You may alternatively declare a Model an EmbeddedDocument (a document stored within another document):

	include MongoMapper::EmbeddedDocument

### Key-Based Data Schema

In MongoDB, data is stored in json-formatted document storage as opposed to a relational database. Because there isn't ActiveRecord or Migrations, use MongoMapper to define our data schema in the model directly.  To do so, make key declarations, like so:

	key :title, String

This would mean that in the model there would be a keypair with the key 'title' and a string value.

### Data Types

MongoDB accepts the following key types:

	*	String
	*	Integer
	*	Time
	* 	Boolean
	* 	Array

To add timestamps to a model:

	timestamps!

### Embedded Documents

MongoDB enables you to store documents within documents, as opposed to having each model be it's own collection of documents. This is useful when a document only belongs to one parent.

For example:

	class Movie
		include MongoMapper::Document
		key :title, String
		many :votes
	end
	
	class Vote
		include MongoMapper::EmbeddedDocument
		key :user_name,      	String
		key :rating,		Integer
	end

## Querying with MongoMapper in Rails 3

Querying with MongoMapper is similiar to querying with ActiveRecord.

###  Find All

To find all documents associated with a model, do:

	ModelName.all

For example, with a model named Movie:

	Movie.all

Would return all movie documents.

###  Find All With Conditions

To find documents with specific conditions, for example all movies made in the year 2011:

	Movie.all(:year => '2011')

### Find First, Last

Find the first record created, for example the first movie document created:

	Movie.first

Or:

	Movie.all(:first)

To find the last:

	Movie.last

Or:

	Movie.all(:last)

### Find Key or Keys

To find records by a specific key, use:

	Model.find_by_key_name(value)

To find the movie Die Hard in our movie example:

	Movie.find_by_title("Die Hard")

To find all records by a specific key, use:

	Model.find_all_by_key_name(value)

To find all records with multiple key values, for example every movie made in 2011 with a running time of 110 minutes:

	Model.find_all_by_year_and_minutes(2011,110)