# faker_fdw

`faker_fdw` is a foreign data wrapper for PostgreSQL that generates fake data. 

## What can I do with it?

You can generate data for testing, create examples for your blog, populate a development database and other things related to generate fake data. 

When `faker_fdw` is installed, create one milion rows on a `people` table is as easy as doing:

```sql
devdb=> INSERT INTO public.people SELECT ssn, name, phone_number FROM faker.people;
```

In the snippet above, `faker.people` is a foreign table that returns some "person" fields, but you can do much more,
like geolocation, addresses, credit card and so on.

## How it works?

It's easy, once `faker_fdw` was installed ([see below how to install](#installing) and create the `faker_srv`), just create 
a foreign table with fields like `ssn`, `name`, `first_name`, `last_name`, `address`, 
`phone_number` and many others. The magic is done by `faker-factory`, a Python library that generate code through [providers](http://fake-factory.readthedocs.org/en/latest/providers.html). For example, `ssn`, `name`, `first_name`, `last_name` are generated by [person provider](http://fake-factory.readthedocs.org/en/latest/providers/faker.providers.person.html#faker-providers-person), while `address` is genereted by [address provider](http://fake-factory.readthedocs.org/en/latest/providers/faker.providers.address.html#faker-providers-address).

We'll start with `ssn`, `name` and `address`:

```sql
guedes=> CREATE FOREIGN TABLE fake.person (ssn varchar, name varchar, address text) 
         SERVER faker_srv OPTIONS (max_results '100');
CREATE FOREIGN TABLE
Time: 36,400 ms
guedes=> SELECT * FROM fake.person limit 10;
     ssn     |        name        |               address                
-------------+--------------------+--------------------------------------
 452-53-4113 | Jordin McClure PhD | 61875 Bernhard Lights Apt. 594      +
             |                    | Shonnamouth, FL 19690-6384
 586-60-9538 | Isabela D'Amore    | 622 Williamson Road                 +
             |                    | Schmidtside, MH 44962
 525-45-7125 | Irving Terry       | 30478 Cummings Turnpike             +
             |                    | New Chazstad, SC 67727-1963
 314-36-1089 | Janette Bradtke    | 73775 Janell Bridge Apt. 120        +
             |                    | Lonzofort, MH 88220
 382-65-0182 | Dayna Lesch        | 71914 Mosciski Fords                +
             |                    | Lake Dalechester, FM 08869-8100
 698-15-6164 | Judie Dickens      | 7634 Leuschke Burgs                 +
             |                    | West Antonio, MD 76638-0668
 870-44-9100 | Brooks Stroman     | 63236 Pfannerstill Junction Apt. 308+
             |                    | South Shea, MS 34801-5187
 652-43-1400 | Kendrick Denesik   | 0077 Runolfsdottir Cape             +
             |                    | Lake Gaynell, RI 42511
 879-63-4746 | Osie Kemmer        | 4343 Jazlynn Knoll                  +
             |                    | Lake Trueport, AL 88273
 837-30-7043 | Nyasia Smitham     | 3043 Gretta Shoal                   +
             |                    | New Haiden, UT 08099-9192
(10 rows)

Time: 17,276 ms
```

Now, lets suppose that you want the `phone_number`, that's easy, just add
the column:

```sql
guedes=> ALTER FOREIGN TABLE fake.person ADD COLUMN phone_number varchar;
ALTER FOREIGN TABLE
Time: 31,200 ms
guedes=> SELECT * FROM fake.person LIMIT 10;
     ssn     |          name           |            address             |    phone_number    
-------------+-------------------------+--------------------------------+--------------------
 520-61-0046 | Miss Nan Hilll DDS      | 1162 Jaron Mill Apt. 435      +| 635.024.1809x351
             |                         | East Isham, UT 99699-8045      | 
 554-47-6145 | Ayden Jenkins DVM       | 41357 McKenzie Skyway         +| 528.396.8357
             |                         | Port Warren, NM 35237          | 
 021-55-5151 | Dr. Randolf McClure PhD | 9738 Prince Corners Suite 091 +| 696.074.2586x2173
             |                         | Harmonhaven, AK 67018          | 
 441-09-6518 | Harlene Hoppe           | 858 Lenard Port               +| 07969004580
             |                         | Kristinafurt, GU 59690         | 
 486-27-1135 | Dr. Amalie Parker       | 5581 Feil Summit Apt. 736     +| 00554249871
             |                         | West Tatiahaven, PR 12346-7661 | 
 681-31-4609 | Louis Aufderhar I       | 216 Hessel Valley Apt. 891    +| (818)010-0501x1646
             |                         | North Dorothea, RI 12275-4420  | 
 419-81-4064 | Lila Koch DVM           | 354 Fay Vista Suite 603       +| 907-143-1119
             |                         | Greenfelderburgh, MI 50297     | 
 308-50-3314 | Toby Shields            | 059 Nitzsche Parks            +| 383.998.7283x035
             |                         | East Daunte, VT 76481          | 
 405-92-2887 | Delwin Lynch            | 467 Carrol Stream Apt. 466    +| +94(9)7970982195
             |                         | Corimouth, AS 08861            | 
 651-62-6328 | Shantell Pfeffer DVM    | 60971 Nya Villages Suite 939  +| 599.631.2393
             |                         | Mamiestad, GU 54537-9334       | 
(10 rows)

Time: 19,201 ms
```

The data are random generated, so each execution even in diferent sessions will generate a new random dataset, but, sometimes, this is not what you want, because your tests must re-run using the same set of data. You can persist data in other tables, rather than select direct from faker tables, or you can set a `seed` option and use the same seed in *different sessions* to get the same set of data, like the following example:

```sql
guedes=> alter foreign table fake.person options ( add seed '1234' );
ALTER FOREIGN TABLE
guedes=> \c - postgres
You are now connected to database "guedes" as user "postgres".
guedes=# select * from fake.person limit 5;
     ssn     |         name          |             address             |     phone_number     
-------------+-----------------------+---------------------------------+----------------------
 165-12-0147 | Dr. Aron Lind IV      | 2564 Julius View               +| 409.956.0720x73661
             |                       | South Amon, MO 14305            | 
 160-16-9547 | Rowan Bauch           | 8474 Thompson Lights           +| 05197913028
             |                       | Schultzburgh, CT 41584          | 
 356-05-2862 | Jaydon Bogisich       | 90204 Sim River Suite 583      +| 663.781.9218x7740
             |                       | Langworthchester, FM 38547-5316 | 
 864-54-9210 | Maximillian Stamm PhD | 8213 Julie Path                +| 1-803-066-7124x72838
             |                       | Port Whitmouth, DC 03464        | 
 721-72-6607 | Foster Schimmel       | USS Ullrich                    +| (423)625-9466
             |                       | FPO AA 79263-9218               | 
(5 rows)

Time: 123.969 ms
guedes=# \c - guedes
You are now connected to database "guedes" as user "guedes".
guedes=> select * from fake.person limit 5;
     ssn     |         name          |             address             |     phone_number     
-------------+-----------------------+---------------------------------+----------------------
 165-12-0147 | Dr. Aron Lind IV      | 2564 Julius View               +| 409.956.0720x73661
             |                       | South Amon, MO 14305            | 
 160-16-9547 | Rowan Bauch           | 8474 Thompson Lights           +| 05197913028
             |                       | Schultzburgh, CT 41584          | 
 356-05-2862 | Jaydon Bogisich       | 90204 Sim River Suite 583      +| 663.781.9218x7740
             |                       | Langworthchester, FM 38547-5316 | 
 864-54-9210 | Maximillian Stamm PhD | 8213 Julie Path                +| 1-803-066-7124x72838
             |                       | Port Whitmouth, DC 03464        | 
 721-72-6607 | Foster Schimmel       | USS Ullrich                    +| (423)625-9466
             |                       | FPO AA 79263-9218               | 
(5 rows)

Time: 115.262 ms
```

## Installing

You must install a few dependencies related to PostgreSQL and 
Python: `multicorn`, `fake-factory` then `fake_fdw`.

In Debian this is as easy as:

```bash
sudo apt-get install postgresql-9.5-python-multicorn
sudo pip install fake-factory
sudo pip install http://github.com/guedes/faker_fdw/archive/master.zip
```

Once packages was installed, choose which database you want `faker_fdw` by typing:

```sql
CREATE EXTENSION multicorn;
CREATE SERVER faker_srv 
 FOREIGN DATA WRAPPER multicorn
 OPTIONS (wrapper 'faker_fdw.FakerForeignDataWrapper');
```

And that's it! Have fun!

## TODO

1. support passing parameters to provider methods like `fake.credit_card_expire(start="now", end="+10y", date_format="%m/%y")` in [credit card provider](http://fake-factory.readthedocs.org/en/latest/providers/faker.providers.credit_card.html#faker-providers-credit-card)
2. extend examples and documentation;
3. create an entire fake schema that return data with integrity between tables, as fake constraints;
4. create a faker_fdw in C;
5. ...

# License

faker_fdw is release under PostgreSQL License.

See [LICENSE](LICENSE) file for information.
