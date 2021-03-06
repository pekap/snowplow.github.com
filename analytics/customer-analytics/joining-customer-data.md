---
layout: section
category: analytics
analytics_category: customer
title: Joining SnowPlow data to other customer data
weight: 3
---

# Joining SnowPlow engagement data with other sources of customer data

The value of data sets increases when they are joined with other, complimentary data sets. No where is this more true than in the case of customer data. Busiensses have for years spend huge sums of money integrating multiple customer data sets (incl. CRM data, financial data, offline sales data, email marketing data etc.) into single datawarehouses and customer views to provide a complete picture of their userbase. Until SnowPlow, however, integrating web analytics with other customer data sources has always been difficult if not impossible. 

In this section, we walk through the process of integrating SnowPlow data with other data sources:

1. [Why integrate web analytics data with other customer data sources](#why)
2. [Necessary ingredients for successful integration](#ingredients)
3. [Uploading customer data into SnowPlow](#upload)
4. [Joining customer data with SnowPlow](#join)

<a name="why" />
## 1. Why integrate web analytics data with other customer data sources?

To date many companies have become very good at using CRM, loyalty and sales data to understand, segment and serve their customer base. Typically, however, they have **not** used web analytics data to understand their customers, even though how users engage with brands and companies through their website and applications provides some of the most revealing data about who a person is, there motivation for engaging with a brand / product / service and how happy they are with that brand / product and service. To illustrate how revealing web analytics data can be, consider:

* Data from online retail sites can reveal what products a user is interested in purchasing, before they potentially visit a store
* Purchase data can also reveal lifestyle interests: for example, someone shoppingi for nappies is most likely to have a young family
* Data from a content or media site can reveal what specific interests a user has
* If a user repeatedly visits a particular product page, but does not buy, it suggests that something is going wrong with the purchase flow, and the user is likely to become frustrated with the service provider. (And potentially stop being a customer)
* Web analytics data might reveal how price sensitive a user is i.e. does their propensity to buy increase dramatically during promotions?
* Web analytics data can be used to distinguish focused buyers (who come on a website looking for a specific product, and zoom in on it quickly) from those who are interested in browsing and then end up buying

By integrating SnowPlow data with other customer data, it is possible learn additional things about your customers, and use that to drive your marketing and platform development decisions.

Potential sources of customer data to integrate with:

* CRM incl. loyalty (e.g. generated by point-of-sale systems, or loyalty cards)
* Marketing databases (e.g. email, direct mail)
* Social media (e.g. Facebook, Twitter)

<a name="ingredients" />
## 2. Necessary ingredients for successful integration

Integrating any two data sources requires that:

* Both sources can be queried via a single interface. (In practice, this often means *moving* one or both source)
* Ability able to join lines of data based on a customer_id

Given the two above requirements, it becomes clear why web analytics data has not been integrated with other customer data sources:

* Web analytics data volumes are typically enormous, making *moving* the data hard. (SnowPlow *fixes* this problem by enabling you to import your data into SnowPlow, rather than export your web analytics data out of it.)
* Web analytics programs are typically bad at reliably identifying users. (SnowPlow *fixes* this problem by exposing a `user_id` and making it striaghtforward to map `user_id`s with `login_id`s from other systems as described in the [identifying-users](identifying-users.html) section.)

<a name="upload" />
## 3. Uploading data into SnowPlow

In order two join two data sets, you need to be able to run a query across both of them. Typically, this means getting copies of them in the same system. There are three options:

1. Export data out of SnowPlow and into the system that the data you want to integrate it with lives e.g. your CRM system
2. Import the data you want to integrate with SnowPlow out the system it usually lives (e.g. your CRM system) and import it into SnowPlow
3. Export data out of SnowPlow and out of the system you want to integrate it with, and import both data sets into a third system to run the join

Whilst all three options are possible, more often then not we recommend companies use option 2 i.e. import the customer data they want to join with SnowPlow into SnowPlow. (Or more specifically, if they are querying SnowPlow data in Hive, import the data into Hive. If they are querying SnowPlow data in Infobright, import the data into Infobright.) The reason is relatively straightforward: SnowPlow data sets tend to be very large: that is why we use either Hive or Infobright to store and query the data, as both these platforms are built for scale. Other sources of customer data e.g. CRM data are typically much smaller scale, so moving them into Hive / Infobright is usually more straightforward then moving SnowPlow data into a non-Hive / non-Infobright data store.

### Importing data into Hive 

If you are running SnowPlow using Hive on Amazon's S3 / EMR infrastructure, uploading data into Hive is a three step process:

1. Export the data from your system in a format suitable for Hive. (In practice, this generally means some kind of text-delimited format e.g. tab-delimited or comma-delimited)
2. Upload the data into S3
3. Create an external table in Hive that references the data, with the correct field definitions

#### 1. Exporting your data in a format suitable for Hive

How you export customer data out of your systems will vary depending on the system. In general, however, nearly all cutomer-data applications will enable you to export customer records as CSV files.

Some useful things to note about the format of the exported CSV file. (If the CSV file is not in the appropriate format, it will need to be transformed before being uploaded into S3):

1. It is important you know the structure of the CSV file, including the column names and data types. This information will be used when you define a new table in Hive that points at this data
2. At least one of the columns should be a user identifier that will be used to do the join
3. If possible, the CSV should not contain the column titles themselves, as this will make importing them into Hive more straightforward

#### 2. Uploading data into S3

Uploading data into S3 is reasonably straight forward: this can be done via the [Amazon Web Services UI][aws-console] or an S3 interface tool like [Bucket Explorer][bucket-explorer].

#### 3. Defining your data in Hive

Once your data is on S3, you need to tell Hive about it, so you can query it. To do that, you'll need to define a Hive table for it:

{% highlight: mysql %}
/* HiveQL / MySQL pseudo-code */
CREATE EXTERNAL TABLE {{table_name}}
(
	field_1 data_type,
	field_2 data_type,
	...
	field_n data_type
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
LINES TERMINATED BY '\n'
LOCATION 's3://LOCATION-BUCKET/LOCATION-PATH/';
{% endhighlight %}

To check that your data has been successfully uploaded, you can run some sample queries within Hive e.g. to view the first 10 rows:

{% highlight: mysql %}
/* HiveQL / MySQL */
SELECT * from {{table_name}} LIMIT 10;
{% endhighlight %}


### Importing your data into Infobright

If you are using Infobright to store and query your SnowPlow data, uploading data into Infobright is easier: you can use any database administration tool that interfaces with MySQL (on which Infobright is based) to create a new table in Infobright and import the CSV in. If the data you are importing lives in another relational database, it may be possible to use a tool like [Navicat][navicat] to move the data directly between one database and the other, or use an ETL tool like [Jitterbit][jitterbit] to manage a regular transfer of data from one database to the other. 

<a name="join" />

Now that both your data sets are available in Hive or Infobright, you are in a position to run an analysis across both set of data.

To do so, however, you need to `join` the two data sets. This requires mapping users, as identified in SnowPlow by the `user_id`, with customers as identified by a `customer_id` in your second data set. This requires firing a SnowPlow event with the `customer_id` at some stage in the customer journey when the ID is available, so that it can be matched against the SnowPlow `user_id`. The most common way of achieving this is to fire the event at login events, as described in the [identifying users with SnowPlow][identifying-users] section of this Cookbook. Assuming this has been done, you can generate the mapping by executing the following SQL query:

{% highlight mysql %}
/* HiveQL / MySQL */
SELECT
user_id,
ev_value AS customer_id
FROM events
WHERE ev_action LIKE 'login'
GROUP BY user_id, ev_value;
{% endhighlight %}

You can then perform queries that join both data tables, the SnowPlow `events` table and your 2nd table of imported customer data:

{% highlight mysql %}
/* HiveQL / MySQL pseudo-code */
SELECT
s.field_1,
s.field_2,
...
e.field_1,
e.field_2
FROM events s
JOIN 2nd-table e
ON s.user_id = e.customer_id;
{% endhighlight %}

You will likely want to aggregate your data by customer, in which case you would `GROUP BY e.customer_id`. (I.e. the ID used in your customer database, which is likely to be more robust than the cookie-based SnowPlow `user_id` for the reasons described in the [identifying users][identifying-users] section).

## Happy integrating SnowPlow data with other sources of customer data?

Find out [how to calculate customer lifetime value][clv] using SnowPlow

[aws-console]: http://console.aws.amazon.com
[bucket-explorer]: http://www.bucketexplorer.com/
[navicat]: http://www.navicat.com/
[jitterbit]: http://www.jitterbit.com/
[identifying-users]: identifying-users.html#login-events
[clv]: customer-lifetime-value.html

