Parts Implemented by Abdullah AKGÜL
===================================

In this section, there are three main tables which are
hotels, expeditions and tickets table, also there two
extra tables which are images and seats table.
The responsibility of these tables belongs to Abdullah AKGÜL.
Attributes of tables which are mentioned above can be seen
in Figure 1.


.. figure:: images/member3/figure1.png
     :scale: 75 %
     :alt: Abdullah AKGÜL's tables

     Figure 1 - Tables that implemented by Abdullah AKGÜL

Hotels Table
------------

hotels table is created for recommending users proper hotels. hotels table is used for store the information about hotels.
The attributes of hotels table are hotel_id, name, email,
description, city, address, phone, website, logo.
hotel_id is primary key for hotels table. There is one more key attribute
that is city. city attribute is foreign key and the reference of the
city attribute is code attribute of the city table.





.. figure:: images/member3/figure1.png
     :scale: 75 %
     :alt: hotels table

     Figure 2 - hotels table

Creation of hotels table and types of attributes of hotels table are given below;

.. code-block:: sql

    CREATE TABLE IF NOT EXISTS hotels
    (
        hotel_id SERIAL NOT NULL PRIMARY KEY,
        name VARCHAR (25) NOT NULL,
        email VARCHAR (50) NOT NULL,
        description VARCHAR (250) NOT NULL,
        city VARCHAR(2),
        address VARCHAR (250) NOT NULL,
        phone VARCHAR (15) NOT NULL,
        website VARCHAR (50),
        logo BYTEA,
        FOREIGN KEY (city) REFERENCES city (code) ON DELETE RESTRICT ON UPDATE CASCADE

    )

Only users that are admin can manipulate the hotels table.

Operations
^^^^^^^^^^

Operations on the hotels table is handled with hotel
class that is given below.

.. code-block:: python

    class Hotel:
        def __init__(self, name, email, description, city, address, phone, website, logo = None):
            self.name = name
            self.email = email
            self.description = description
            self.city = city
            self.address = address
            self.phone = phone
            self.website = website
            self.logo = logo

This class corresponds the hotels table in the database.
The attributes are same with hotel table.
This class provides ease on operations on the hotels table.


The operations on the hotels table are handled with given below class.
With this class database connection is provided and operations are handled with
functions of this class

.. code-block:: python

    class Hotel:
        def __init__(self):
            if os.getenv("DATABASE_URL") is None:
                self.url = "postgres://itucs:itucspw@localhost:32768/itucsdb"
            else:
                self.url = os.getenv("DATABASE_URL")




Operations on the hotels table is listed below.



Insert
______

Insertion of hotel on hotels table can be performed with two ways.
The one of inserting is inserting hotel without logo attribute.
With this way, logo attribute will be NULL.
Related function is given below as add_hotel.
Other way is inserting hotel with logo attribute. This method is given below as add_hotel_with_logo.
The data for logo is provided with given below code.

.. code-block:: python

    logo = request.files["logo"].read()

These functions takes hotel parameter which is hotel class.

.. code-block:: python

    def add_hotel(self, hotel):
        with dbapi2.connect(self.url) as connection:
            cursor = connection.cursor()
            cursor.execute(
                "INSERT INTO hotels ( name, email, description, city, address, phone, website) VALUES (%s, %s, %s, %s, %s, %s, %s)",
                (hotel.name, hotel.email, hotel.description, hotel.city, hotel.address, hotel.phone, hotel.website))
            cursor.close()

    def add_hotel_with_logo(self, hotel_with_logo):
        with dbapi2.connect(self.url) as connection:
            cursor = connection.cursor()
            cursor.execute(
                "INSERT INTO hotels ( name, email, description, city, address, phone, website, logo) VALUES (%s, %s, %s, %s, %s, %s, %s, %s)",
                (hotel_with_logo.name, hotel_with_logo.email, hotel_with_logo.description, hotel_with_logo.city, hotel_with_logo.address, hotel_with_logo.phone, hotel_with_logo.website, hotel_with_logo.logo))
            cursor.close()

With this insertion functions new hotel will be added as a row to hotels table.

Read
____

There are three different methods for reading data from
hotels table. These methods are given below.


.. code-block:: python

        def get_hotel(self, hotel_id):
            _hotel = None
            try:
                connection = dbapi2.connect(self.url)
                cursor = connection.cursor()
                cursor.execute("SELECT * FROM hotels WHERE hotel_id = %s", (hotel_id,))
                hotel = cursor.fetchone()
                if hotel is not None:
                    _hotel = Hotel(hotel[1], hotel[2], hotel[3], hotel[4], hotel[5], hotel[6], hotel[7], hotel[8])
                connection.commit()
                cursor.close()
            except (Exception, dbapi2.DatabaseError) as error:
                print(error)
            finally:
                if connection is not None:
                    connection.close()
            return _hotel

        def get_hotels(self):
            hotels = []
            try:
                connection = dbapi2.connect(self.url)
                cursor = connection.cursor()
                cursor.execute("SELECT * FROM hotels;")
                for hotel in cursor:
                    _hotel = Hotel(hotel[1], hotel[2], hotel[3], hotel[4], hotel[5], hotel[6], hotel[7], hotel[8])
                    hotels.append((hotel[0], _hotel))
                connection.commit()
                cursor.close()
            except (Exception, dbapi2.DatabaseError) as error:
                print(error)
            finally:
                if connection is not None:
                    connection.close()
            return hotels

        def get_hotels_with_cities(self):
            hotels = []
            try:
                connection = dbapi2.connect(self.url)
                cursor = connection.cursor()
                cursor.execute("SELECT hotel_id, city_name FROM hotels JOIN city ON hotels.city = city.code;")
                hotels = cursor.fetchall()
                connection.commit()
                cursor.close()
            except (Exception, dbapi2.DatabaseError) as error:
                print(error)
            finally:
                if connection is not None:
                    connection.close()
            return hotels


get_hotel method takes hotel_id as parameter. This method
simply returns desired hotel as hotel class.

get_hotels method is used for returning whole hotels in hotels table.
This methods returns an array that created with tuple
which is hotel_id, hotel as hotel class.

get_hotels_with_cities method is nearly same with get_hotels.
The difference is that get_hotels_with_cities returns whole hotels
in the hotels table with city names by using JOIN with city table.

The logo of the hotel is stored as BLOB. For showing logo
as picture format, the data of the logo decoded with given code below.

.. code-block:: python

    from base64 import b64encode

    logo = b64encode(temp_hotel.logo).decode("utf-8")

Update
______

Update hotel operation can be handled with given code below;

.. code-block:: python

        def update_hotel(self, hotel_id, hotel):
            try:
                connection = dbapi2.connect(self.url)
                cursor = connection.cursor()
                cursor.execute("""UPDATE hotels SET name = %s, email = %s, description = %s, city = %s, address = %s, phone = %s, website = %s WHERE hotel_id = %s """, (hotel.name, hotel.email, hotel.description, hotel.city, hotel.address, hotel.phone, hotel.website, hotel_id))
                connection.commit()
                cursor.close()
            except (Exception, dbapi2.DatabaseError) as error:
                print(error)
            finally:
                if connection is not None:
                    connection.close()

        def update_hotel_with_logo(self, hotel_id, hotel):
            try:
                connection = dbapi2.connect(self.url)
                cursor = connection.cursor()
                cursor.execute("""UPDATE hotels SET name = %s, email = %s, description = %s, city = %s, address = %s, phone = %s, website = %s, logo = %s WHERE hotel_id = %s """, (hotel.name, hotel.email, hotel.description, hotel.city, hotel.address, hotel.phone, hotel.website, hotel.logo, hotel_id))
                connection.commit()
                cursor.close()
            except (Exception, dbapi2.DatabaseError) as error:
                print(error)
            finally:
                if connection is not None:
                    connection.close()

As seen on code, there are two method for updating hotel table.

update_hotel method takes hotel_id and hotel class as parameter.
This method updates the hotel whose hotel_id is equal to taken hotel_id
with taken hotel class attributes but without logo attribute.

update_hotel_with_logo method takes hotel_id and hotel class as parameter.
This method updates the hotel whose hotel_id is equal to taken hotel_id
with taken hotel class attributes.

After update operations, hotel table will be updated.

Delete
______

Delete operation is handled with given code below;

.. code-block:: python


    def delete_hotel(self, hotel_id):
        try:
            connection = dbapi2.connect(self.url)
            cursor = connection.cursor()
            cursor.execute("DELETE FROM hotels WHERE hotel_id = %s", (hotel_id,))
            connection.commit()
            cursor.close()
        except (Exception, dbapi2.DatabaseError) as error:
            print(error)
        finally:
            if connection is not None:
                connection.close()


    def delete_hotel_logo(self, hotel_id):
        try:
            connection = dbapi2.connect(self.url)
            cursor = connection.cursor()
            cursor.execute("UPDATE hotels SET logo = NULL WHERE hotel_id = %s", (hotel_id,))
            connection.commit()
            cursor.close()
        except (Exception, dbapi2.DatabaseError) as error:
            print(error)
        finally:
            if connection is not None:
                connection.close()

The deletion of hotel is handled with delete_hotel method. The selected hotel
will be deleted in hotels table by matching hotel_id taken as parameter.

The logo of hotel can be deleted without deleting the whole hotel information with
delete_hotel_logo method.
After delete_hotel_logo method, logo of the hotel will be NULL. The deletion of logo
is provided with matching hotel_id taken as parameter to this method.


Search
______

The search operation on hotel table is handled with given code below;

.. code-block:: python

    def search(self, text):
        hotels = []
        to_search = "%" + text + "%"
        try:
            connection = dbapi2.connect(self.url)
            cursor = connection.cursor()
            cursor.execute("SELECT * FROM hotels JOIN city ON city.code = hotels.city WHERE (LOWER(name) like LOWER(%s)) or (LOWER(email) like LOWER(%s)) or (LOWER(description) like LOWER(%s)) or (LOWER(address) like LOWER(%s)) or (LOWER(website) like LOWER(%s)) or (LOWER(city_name) like LOWER(%s))    ;", (to_search, to_search, to_search, to_search, to_search, to_search))
            for hotel in cursor:
                _hotel = Hotel(hotel[1], hotel[2], hotel[3], hotel[4], hotel[5], hotel[6], hotel[7], hotel[8])
                hotels.append((hotel[0], _hotel))
            connection.commit()
            cursor.close()
        except (Exception, dbapi2.DatabaseError) as error:
            print(error)
        finally:
            if connection is not None:
                connection.close()
        return hotels


The search method takes text as string. This string is searched in whole hotels table join with city table on city.
To search with case-insensitive string and whole data in hotels table
is used with LOWER function. This method returns array of tuple that has hotel_id and hotel
that has that string in anywhere on hotel information.


Expeditions Table
-----------------

expeditions table is created for providing expeditions to users by firms.
expeditions table is used for store the information about expeditions.
The attributes of expeditions table are expedition_id, from_city, from_ter,
to_city, to_ter, dep_time, arr_time, date, price, plane_id, current_cap, total_cap,
total_cap, driver_id, firm_id and document.
firm_id is primary key for hotels table.
There are six more key attributes
that are from_city, to_city, from_ter, to_ter, plane_id and driver_id.
from_city and to_city attributes are foreign keys and the reference of the
from_city and to_city attributes is code attribute of the city table.
from_ter and to_ter attributes are foreign keys and the reference of the
from_ter and to_ter attributes is terminal_id attribute of the terminal table.
plane_id is foreign key and the reference of the
plane_id attribute is vehicle_id attribute of the vehicles table.
driver_id is foreign key and the reference of the
driver_id attribute is firm_id attribute of the firms table.
Also total_cap attribute comes from
vehicle tables capacity attribute but it is not foreign key because the capacity attribute of
vehicles table is not primary key.




.. figure:: images/member3/figure1.png
     :scale: 75 %
     :alt: expeditions table

     Figure 3 - expeditions table

Creation of expeditions table and types of attributes of hotels table are given below;

.. code-block:: sql

    CREATE TABLE IF NOT EXISTS expeditions
    (
        expedition_id SERIAL NOT NULL PRIMARY KEY,
        from_city VARCHAR (02) NOT NULL,
        from_ter INT NOT NULL,
        to_city VARCHAR (02) NOT NULL,
        to_ter INT NOT NULL,
        dep_time VARCHAR (5) NOT NULL ,
        arr_time VARCHAR (5) NOT NULL ,
        date VARCHAR (10) NOT NULL ,
        price INT NOT NULL CHECK (price >= 10),
        plane_id INT NOT NULL ,
        current_cap INT NOT NULL DEFAULT 0,
        total_cap INT NOT NULL,
        driver_id INT NOT NULL,
        firm_id INT NOT NULL,
        document BYTEA,
        FOREIGN KEY (from_city) REFERENCES city (code) ON DELETE RESTRICT ON UPDATE CASCADE,
        FOREIGN KEY (to_city) REFERENCES city (code) ON DELETE RESTRICT ON UPDATE CASCADE,
        FOREIGN KEY (from_ter) REFERENCES terminal (terminal_id) ON DELETE RESTRICT ON UPDATE CASCADE,
        FOREIGN KEY (to_ter) REFERENCES terminal (terminal_id) ON DELETE RESTRICT ON UPDATE CASCADE,
        FOREIGN KEY (plane_id) REFERENCES vehicles (vehicle_id) ON DELETE RESTRICT ON UPDATE CASCADE,
        FOREIGN KEY (driver_id) REFERENCES drivers (driver_id) ON DELETE RESTRICT ON UPDATE CASCADE,
        FOREIGN KEY (firm_id) REFERENCES firms (firm_id) ON DELETE RESTRICT ON UPDATE CASCADE


    )

Only users that are admin and firms can manipulate the expeditions table.

Operations
^^^^^^^^^^

Operations on the expeditions table is handled with expedition
class that is given below.

.. code-block:: python

    class Expedition:
        def __init__(self, from_, from_ter, to, to_ter, dep_time, arr_time, date, price, selected_plane, driver_id, firm_id, total_cap, current_cap = 0, document = None):
            self.from_ = from_
            self.from_ter = from_ter
            self.to = to
            self.to_ter = to_ter
            self.dep_time = dep_time
            self.arr_time = arr_time
            self.date = date
            self.price = price
            self.selected_plane = selected_plane
            self.driver_id = driver_id
            self.firm_id = firm_id
            self.total_cap = total_cap
            self.current_cap = current_cap
            self.document = document

This class corresponds the expeditions table in the database.
The attributes are same with expeditions table.
This class provides ease on operations on the expeditions table.


The operations on the expeditions table are handled with given below class.
With this class database connection is provided and operations are handled with
functions of this class

.. code-block:: python

    class Expedition:
        def __init__(self):
            if os.getenv("DATABASE_URL") is None:
                self.url = "postgres://itucs:itucspw@localhost:32768/itucsdb"
            else:
                self.url = os.getenv("DATABASE_URL")


Operations on the expeditions table is listed below.



Insert
______

Insertion of expedition on expeditions table can be performed with two ways.
The one of inserting is inserting expedition without document attribute.
With this way, document attribute will be NULL.
Related function is given below as add_expedition.
Other way is inserting expedition with document attribute. This method is given below as add_expedition_with_document.
The data for document is provided with given below code.

.. code-block:: python

    document = request.files["document"].read()

These functions takes expedition parameter which is expedition class.

.. code-block:: python

    def add_expedition(self, expedition):
        with dbapi2.connect(self.url) as connection:
            cursor = connection.cursor()
            cursor.execute(
                "INSERT INTO expeditions ( from_city, from_ter, to_city, to_ter, dep_time, arr_time, date, price, plane_id, firm_id, total_cap, current_cap, driver_id) VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s)",
                (expedition.from_, expedition.from_ter, expedition.to, expedition.to_ter, expedition.dep_time, expedition.arr_time, expedition.date, expedition.price, expedition.selected_plane, expedition.firm_id,expedition.total_cap, expedition.current_cap, expedition.driver_id))
            cursor.close()

    def add_expedition_with_document(self, expedition):
        with dbapi2.connect(self.url) as connection:
            cursor = connection.cursor()
            cursor.execute(
                "INSERT INTO expeditions ( from_city, from_ter, to_city, to_ter, dep_time, arr_time, date, price, plane_id, firm_id, total_cap, current_cap, driver_id, document) VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s)",
                (expedition.from_, expedition.from_ter, expedition.to, expedition.to_ter, expedition.dep_time, expedition.arr_time, expedition.date, expedition.price, expedition.selected_plane, expedition.firm_id, expedition.total_cap, expedition.current_cap, expedition.driver_id, expedition.document ))
            cursor.close()

Foreign key's information are coming from other related tables.
With this insertion functions new expedition will be added as a row to expeditions table.

Read
____

There are five different methods for reading data from
expeditions table. These methods are given below.


.. code-block:: python

        def get_all_valid_expeditions(self):
            expeditions = []
            try:
                connection = dbapi2.connect(self.url)
                cursor = connection.cursor()
                cursor.execute("SELECT * FROM expeditions where current_cap < total_cap;")
                for expedition in cursor:
                    _expedition = Expedition(expedition[1], expedition[2], expedition[3], expedition[4], expedition[5],
                                             expedition[6], expedition[7], expedition[8], expedition[9], expedition[12],
                                             expedition[13], expedition[11], expedition[10], expedition[14])
                    _expedition.expedition_id  =expedition[0]
                    if dayCompare(_expedition.date):
                        expeditions.append((expedition[0], _expedition))
                cursor.close()
            except (Exception, dbapi2.DatabaseError) as error:
                print(error)
            finally:
                if connection is not None:
                    connection.close()
            return expeditions

        def get_all_expeditions(self):
            expeditions = []
            try:
                connection = dbapi2.connect(self.url)
                cursor = connection.cursor()
                cursor.execute("SELECT * FROM expeditions ;")
                for expedition in cursor:
                    _expedition = Expedition(expedition[1], expedition[2], expedition[3], expedition[4], expedition[5],
                                             expedition[6], expedition[7], expedition[8], expedition[9], expedition[12],
                                             expedition[13], expedition[11], expedition[10], expedition[14])
                    _expedition.expedition_id  =expedition[0]

                    expeditions.append((expedition[0], _expedition))
                cursor.close()
            except (Exception, dbapi2.DatabaseError) as error:
                print(error)
            finally:
                if connection is not None:
                    connection.close()
            return expeditions

        def get_filtered_expeditions(self, to_city, to_ter, from_city, from_ter, firm_id, date, max_price):
            expeditions = []
            statement = " SELECT * FROM expeditions WHERE TRUE  "

            if to_city is not None:
                statement += " and to_city = '" + to_city + "' "
            if to_ter is not None:
                statement += "and to_ter = " + str(to_ter) + " "
            if from_city is not None:
                statement += " and from_city = '" + from_city + "' "
            if from_ter is not None:
                statement += " and from_ter = " + str(from_ter) + " "
            if firm_id is not None:
                statement += "and firm_id = " + str(firm_id) + " "
            if date is not "":
                statement += "and date like '%" + date + "%' "
            if max_price is not "":
                statement += "and price <= " + str(max_price)
            statement += "and current_cap < total_cap"
            try:
                connection = dbapi2.connect(self.url)
                cursor = connection.cursor()
                cursor.execute(statement)
                for expedition in cursor:
                    _expedition = Expedition(expedition[1], expedition[2], expedition[3], expedition[4], expedition[5],
                                             expedition[6], expedition[7], expedition[8], expedition[9], expedition[12],
                                             expedition[13], expedition[11], expedition[10], expedition[14])
                    _expedition.expedition_id  =expedition[0]
                    if dayCompare(_expedition.date):
                        expeditions.append((expedition[0], _expedition))
                cursor.close()
            except (Exception, dbapi2.DatabaseError) as error:
                print(error)
            finally:
                if connection is not None:
                    connection.close()

            return expeditions

        def get_firms_expedition(self, firm_id):
            expeditions = []
            try:
                connection = dbapi2.connect(self.url)
                cursor = connection.cursor()
                cursor.execute("SELECT * FROM expeditions WHERE firm_id = %s;", (firm_id,))
                for expedition in cursor:
                    _expedition = Expedition(expedition[1], expedition[2], expedition[3], expedition[4], expedition[5],
                                             expedition[6], expedition[7], expedition[8], expedition[9], expedition[12],
                                             expedition[13], expedition[11], expedition[10], expedition[14])
                    expeditions.append((expedition[0], _expedition))
                connection.commit()
                cursor.close()
            except (Exception, dbapi2.DatabaseError) as error:
                print(error)
            finally:
                if connection is not None:
                    connection.close()
            return expeditions


        def get_expedition(self, expedition_id):
            _expedition = None
            try:
                connection = dbapi2.connect(self.url)
                cursor = connection.cursor()
                cursor.execute("SELECT * FROM expeditions WHERE expedition_id = %s", (expedition_id,))
                expedition = cursor.fetchone()
                if expedition is not None:
                    _expedition = Expedition(expedition[1], expedition[2], expedition[3], expedition[4], expedition[5],
                                             expedition[6], expedition[7], expedition[8], expedition[9], expedition[12],
                                             expedition[13], expedition[11], expedition[10], expedition[14])
                connection.commit()
                cursor.close()
            except (Exception, dbapi2.DatabaseError) as error:
                print(error)
            finally:
                if connection is not None:
                    connection.close()
            return _expedition


get_all_valid_expeditions returns whole expeditions that are up-to-date and
have empty seat. Empty seat checks whether current_cap attribute is smaller
than total_cap. The up-to-date checking is done by dayCompare function that is given below;

.. code-block:: python

    from datetime import datetime

    today = datetime.today()

    str_today = str(today.month) + '/' + str(today.day) + '/' + str(today.year)

    def dayCompare( toCompare):
        t0 = str_today.split('/', 3)
        t1 = toCompare.split('/', 3)
        if t0[2] > t1[2]:
            return False
        elif t0[2] == t1[2]:
            if t0[1] > t1[1]:
                return False
            elif t0[1] == t1[1]:
                if t0[0] >= t1[0]:
                    return False
                else:
                    return True
            else:
                return True
        else:
            return True

This dayCompare compare function takes the expedition's date attribute as parameter.
Then compare it with today's date with desired format.
After checking, get_all_valid_expeditions method returns array of tuples that has
expedition_id and expedition that is expedition class.

get_all_expeditions method is used for returns whole expeditions in expeditions table.
This methods returns an array that created with tuple
which is expedition_id, expedition as expedition class.

get_filtered_expeditions method is filter the expeditions.
The filter is done with taken parameters that are to_city, o_city, to_ter, from_city, from_ter, firm_id, date and max_price .
This methods returns an array that created with tuple
which is expedition_id, expedition as expedition class.

get_firms_expedition method takes firm_id as parameter.
This method used for getting the firm's expeditions by matching firm_id.
This methods returns an array that created with tuple
which is expedition_id, expedition as expedition class.

get_expedition method takes expedition_id as parameter.
get_expedition method is used for returning desired expedition
by matching expedition_id.




The document of the expedition is stored as BLOB. For showing document
as PDF format, the data of the document handled with given code below.

.. code-block:: python


    from flask import send_file
    from io import BytesIO

    file_data = expedition_db.get_expedition(expedition_id).document
    file_name = str(expedition_id) + '.pdf'
    return send_file(BytesIO(file_data), attachment_filename = file_name, as_attachment=True)

Update
______

Update expedition operation can be handled with given code below;

.. code-block:: python

        def update_expedition(self, expedition_id, expedition):
            with dbapi2.connect(self.url) as connection:
                cursor = connection.cursor()
                cursor.execute(
                    "UPDATE expeditions SET from_city = %s, from_ter = %s, to_city = %s, to_ter = %s, dep_time = %s, arr_time = %s, date = %s, price = %s, plane_id = %s, firm_id = %s, total_cap = %s, current_cap = %s, driver_id = %s WHERE expedition_id = %s",
                    (expedition.from_, expedition.from_ter, expedition.to, expedition.to_ter, expedition.dep_time, expedition.arr_time, expedition.date, expedition.price, expedition.selected_plane, expedition.firm_id,expedition.total_cap, expedition.current_cap, expedition.driver_id, expedition_id))
                cursor.close()

        def update_expedition_with_document(self, expedition_id, expedition):
            with dbapi2.connect(self.url) as connection:
                cursor = connection.cursor()
                cursor.execute(
                    "UPDATE expeditions SET from_city = %s, from_ter = %s, to_city = %s, to_ter = %s, dep_time = %s, arr_time = %s, date = %s, price = %s, plane_id = %s, firm_id = %s, total_cap = %s, current_cap = %s, driver_id = %s, document = %s WHERE expedition_id = %s",
                    (expedition.from_, expedition.from_ter, expedition.to, expedition.to_ter, expedition.dep_time, expedition.arr_time, expedition.date, expedition.price, expedition.selected_plane, expedition.firm_id, expedition.total_cap, expedition.current_cap, expedition.driver_id, expedition.document, expedition_id))
                cursor.close()

As seen on code, there are two method for updating expeditions table.

update_expedition method takes expedition_id and expedition class as parameter.
This method updates the expedition whose expedition_id is equal to taken expedition_id
with taken expedition class attributes but without document attribute.

update_expedition_with_document method takes expedition_id and expedition class as parameter.
This method updates the expedition whose expedition_id is equal to taken expedition_id
with taken expedition class attributes.

After update operations, expeditions table will be updated.

Delete
______

Delete operation is handled with given code below;

.. code-block:: python


        def delete_expedition(self, expedition_id):
            try:
                connection = dbapi2.connect(self.url)
                cursor = connection.cursor()
                cursor.execute("DELETE FROM expeditions WHERE expedition_id = %s", (expedition_id,))
                connection.commit()
                cursor.close()
            except (Exception, dbapi2.DatabaseError) as error:
                print(error)
            finally:
                if connection is not None:
                    connection.close()

        def delete_expedition_document(self, expedition_id):
            try:
                connection = dbapi2.connect(self.url)
                cursor = connection.cursor()
                cursor.execute("UPDATE expeditions SET document = NULL WHERE expedition_id = %s", (expedition_id,))
                connection.commit()
                cursor.close()
            except (Exception, dbapi2.DatabaseError) as error:
                print(error)
            finally:
                if connection is not None:
                    connection.close()


The deletion of expedition is handled with delete_expedition method. The selected expedition
will be deleted in expeditions table by matching expedition_id taken as parameter.

The document of expedition can be deleted without deleting the whole expedition information with
delete_expedition_document method.
After delete_expedition_document method, document of the expedition will be NULL. The deletion of document
is provided with matching expedition_id taken as parameter to this method.


Search
______

The search operation on expedition table is handled with given code below;

.. code-block:: python

    def search(self, text):
        expeditions = []
        to_search = "%" + text + "%"
        try:
            connection = dbapi2.connect(self.url)
            cursor = connection.cursor()
            if isInt(text):

                cursor.execute("""select * from expeditions where expedition_id in (
                                select expedition_id
                                from expeditions, city as to_city, firms, city as from_city, terminal as to_ter, terminal as from_ter
                                where (firms.firm_id = expeditions.firm_id and expeditions.to_city = to_city.code and expeditions.from_city = from_city.code and expeditions.to_ter = to_ter.terminal_id and expeditions.from_ter = from_ter.terminal_id )
                                and
                                ( (price = %s) or (LOWER(to_city.city_name) like LOWER(%s)) or ( LOWER(firms.name) like LOWER(%s) ) or ( LOWER(from_city.city_name) like LOWER(%s) ) or (LOWER(date) like LOWER(%s)) or (LOWER(dep_time) like LOWER(%s)) or (LOWER(arr_time) like LOWER(%s)) or (LOWER(from_ter.terminal_name) like LOWER(%s)) or (LOWER(to_ter.terminal_name) like LOWER(%s))))""", (int(text) ,to_search, to_search, to_search, to_search, to_search, to_search,to_search,to_search, ))
            else:
                cursor.execute("""select * from expeditions where expedition_id in (
                                select expedition_id
                                from expeditions, city as to_city, firms, city as from_city, terminal as to_ter, terminal as from_ter
                                where (firms.firm_id = expeditions.firm_id and expeditions.to_city = to_city.code and expeditions.from_city = from_city.code and expeditions.to_ter = to_ter.terminal_id and expeditions.from_ter = from_ter.terminal_id )
                                and
                                (  (LOWER(to_city.city_name) like LOWER(%s)) or ( LOWER(firms.name) like LOWER(%s) ) or ( LOWER(from_city.city_name) like LOWER(%s) ) or (LOWER(date) like LOWER(%s)) or (LOWER(dep_time) like LOWER(%s)) or (LOWER(arr_time) like LOWER(%s)) or (LOWER(from_ter.terminal_name) like LOWER(%s)) or (LOWER(to_ter.terminal_name) like LOWER(%s))))""",
                               ( to_search, to_search, to_search, to_search, to_search, to_search, to_search,
                                to_search,))

            for expedition in cursor:

                _expedition = Expedition(expedition[1], expedition[2], expedition[3], expedition[4], expedition[5],
                                         expedition[6], expedition[7], expedition[8], expedition[9], expedition[12],
                                         expedition[13], expedition[11], expedition[10], expedition[14])
                if dayCompare(_expedition.date):
                    expeditions.append((expedition[0], _expedition))
            connection.commit()
            cursor.close()
        except (Exception, dbapi2.DatabaseError) as error:
            print(error)
        finally:
            if connection is not None:
                connection.close()
        return expeditions



The search method takes text as string. This string is searched in whole expeditions table join with related tables.
To search with case-insensitive string and whole data in expeditions table
is used with LOWER function.
Also the code checks whether given string can be integer or not. If given string can be integer
code will be search on price too. The checking code is given below.
This method returns array of tuple that has expedition_id and expedition
that has that string in anywhere on expedition information.

.. code-block:: python

    def isInt(value):
      try:
        int(value)
        return True
      except ValueError:
        return False


Related Systems
^^^^^^^^^^^^^^^

The usage of the expeditions table are listed below.


Buy Ticket
__________

When a user buy a ticket, expedition table should be updated. This updating is given below.

.. code-block:: python

    def bought(self, expedition_id ):
        with dbapi2.connect(self.url) as connection:
            cursor = connection.cursor()
            cursor.execute(
                "UPDATE expeditions SET current_cap = current_cap + 1 WHERE expedition_id = %s",
                (expedition_id, ))
            cursor.close()


bought method takes expedition_id as parameter. When user buy a ticket for an expedition,
current_cap attribute of that expedition should be incremented by one. bought method updates
the expedition table with this purpose.

Cancel Ticket
_____________

When a user cancel a ticket, expedition table should be updated. This updating is given below.


.. code-block:: python


    def cancelled(self, expedition_id):
        with dbapi2.connect(self.url) as connection:
            cursor = connection.cursor()
            cursor.execute(
                "UPDATE expeditions SET current_cap = current_cap - 1 WHERE expedition_id = %s",
                (expedition_id,))
            cursor.close()

cancelled method takes expedition_id as parameter. When user cancel a ticket for an expedition,
current_cap attribute of that expedition should be decremented by one. cancelled method updates
the expedition table with this purpose.
