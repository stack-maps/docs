---
title: Stack Map Reference Documentation

toc_footers:
  - <a href='https://github.com/tripit/slate'>Documentation Powered by Slate</a>

search: true
---

# Introduction

This is the documentation for Stack Maps, a project aimed at reducing time spent at find a book in a library. This documentation is designed for people maintaining the system. It describes the overall system architecture, database design, exposed API, and data transport formats.

[stack-maps/server](https://github.com/stack-maps/server)
[stack-maps/map-editor](https://github.com/stack-maps/map-editor)

# Requirements

* A server that supports php 7.0.
* MySQL database.

# Deployment

1. Fork the server repository.
2. Clone the forked repository to local disk.
3. Locate `db-setup.sql` file inside the repository. In your MySQL database, run `db-setup.sql`. This will create all tables necessary for the system. By default the script does not create a new database. You can change this by uncommenting lines 20-21 in `db-setup.sql`.
4. Locate `api.php` file inside the repository. Replace lines 9-13 with your database information. Upload `api.php` file on your server, preferrably on the same server as your database.
5. Locate `displaymap.js` file inside the repository. Replace line 141 with your 'api.php' file path. This can be an absolute path, relative path or an internet address. Upload `displaymap.js` and `index.html` files on your server in the same folder.

The system is now up and running! Access the `index.html` file to see how you would integrate the system into your `library` catalog page. Download the newest release from [stack-maps/map-editor](https://github.com/stack-maps/map-editor) to start populating your library, or write your own following this API.

# Architecture Overview

## Database

This project uses MySQL database. While any remote MySQL database can work, for security concerns it is the best to have the MySQL database and the access script on the same server, and disallow any remote access on the database.

## Access API
The access API acts as the getter/setter of the database. It provides an extra layer of security as it can deny attacks on the database. For this project, the access API comes in the form of a php script. It will need host, port, username and password of the database. To set up the API, simply replace those information at the top of the file, then upload it to a server.

Custom security authorization logic should be added to this script to allow or deny database updating API calls. See the file for details on how to do so.

## Map
The map is what `library` patrons see when they click on a button. We wrap the map in a javascript file and provide a sample HTML page. To set this up, the javascript file should be uploaded to the same directory of wherever the actual MAP button is in the `library` catalogs, and in that HTML page, this javascript file should be imported, and the `displayMap(callno)` function should be called when the button is pressed.

## Map Editor
Once the server set up is complete, we need to create some library floor plans to actually use this system. While inputing [here](https://github.com/stack-maps/map-editor).

# JSON Format

## library

```json
{
  "library_id": 1,
  "library_name": "Uris",
  "floors": []
}
```

Name | Type
--------- | -----------
`library_id`  | int
`library_name`  | string
`floors` | array of [floor](#floor) objects

## floor

```json
{
  "floor_id": 1,
  "floor_name": "Level 1",
  "floor_order": 1,
  "walls": [],
  "aisle_areas": [],
  "landmarks": [],
  "library": 1
}
```

Name | Type
--------- | -----------
`floor_id`  | int
`floor_name`  | string
`floor_order` | float
`walls` | array of [wall](#wall) objects
`aisle_areas` | array of [aisle_area](#aisle_area) objects
`aisles` | array of [aisle](#aisle) objects
`landmarks` | array of [landmark](#landmark) objects
`library` | int

`library` refers to the library_id, which this `floor` belongs to..

## wall

```json
{
  "start_x": 15.3,
  "start_y": 16.8,
  "end_x"  : 30.6,
  "end_y"  : 33.6,
  "floor"  : 1
}
```

Name | Type
--------- | -----------
`wall_id`  | int
`start_x`  | float
`start_y` | float
`end_x` | float
`end_y` | float
`floor` | int

`floor` refers to the floor_id, which this wall belongs to.

## aisle_area

```json
{
  "center_x": 15.3,
  "center_y": 16.8,
  "width"  : 30.6,
  "height"  : 33.6,
  "rotation"  : 36,
  "aisles" : []
}
```

Name | Type
--------- | -----------
`aisle_area_id`  | int
`center_x`  | float
`center_y` | float
`width` | float
`height` | float
`rotation` | float
`aisles` | array of [aisle](#aisle) objects
`floor` | int

`floor` refers to the `floor_id`, which this `aisle_area` belongs to.

## aisle

```json
{
  "center_x": 15.3,
  "center_y": 16.8,
  "length"  : 30.6,
  "width"  : 33.6,
  "rotation"  : 16,
  "sides" : 2,
  "call-range": []
}
```

Name | Type
--------- | -----------
`aisle_id`  | int
`center_x`  | float
`center_y` | float
`width` | float
`height` | float
`rotation` | float
`is_double_sided` | boolean
`call_ranges` | array of [call_range](#call_range) objects
`aisle_area` | int
`floor` | int

`aisle_area` refers to the `aisle_area_id`, which this aisle belongs to.
`floor` refers to the floor_id, which this aisle belongs to.

## call_range

```json
{
  "collection": "OVER",
  "call_start": "E",
  "call_end"  : "F",
  "side"  : 2,
}
```

Name | Type
--------- | -----------
`call_range_id`  | int
`collection`  | string
`call_start` | string
`call_end` | string
`side` | int
`aisle` | int

`aisle` refers to the `aisle_id`, which this `call_range` belongs to.

## landmark

```json
{
  "landmark_type" : "toilet",
  "center_x": 15.3,
  "center_y": 16.8,
  "length"  : 30.6,
  "width"  : 33.6,
  "rotation"  : 16
}
```

Name | Type
--------- | -----------
`landmark_id`  | int
`landmark_type`  | string
`center_x` | float
`center_y` | float
`width` | float
`height` | float
`rotation` | float
`floor` | int

`floor` refers to the `floor_id`, which this `landmark` belongs to.

# API Functions

The following are key functions provided by the API to allow interaction between the Map editor, Map generator and the database.

They are called using POST and responses are formatted as a Json object specified above.

## login

**Description:** This fetches username and password information from the form, then runs custom credential checks. If successful, it creates a new token in the database and returns it to the client.

Custom credential checks routed to third party should be executed here.

### POST Parameters

Parameter | Description
--------- | -----------
`username`  | Username entered by the client.
`password`  | Password entered by the client.

### Response Format

Parameter | Description
--------- | -----------
`success`  | Whether login was successful.
`error`    | The error message, if not successful.
`token`    | The token for this login, if successful. This is used as quick credential checks for subsequent db modifications.

## create_library

This creates a `library` in the database and returns a success flag along with the created `library` id.

### POST Parameters

Parameter       | Description
---------       | -----------
`token`        | Access token given from login.
`library_name`  | Name of the new library. This must be different from existing names in the database.

### Response Format

Parameter  | Description
---------  | -----------
`success`  | Whether creation was successful.
`error`    | The error message, if not successful.
`id`       | The library_id of the newly created `library` record in the database.

## create_floor

This creates a `floor` in the database and returns a success flag along with the created `floor` id. The newly created `floor` is empty.

### POST Parameters

Parameter       | Description
---------       | -----------
`token`         | Access token given from login.
`floor_name`    | Name of the new library. This must be different from existing names in the database.
`floor_order`   | Order of the new floor. Essentially a number that serves to order the floors. See database design document for details on how this works.
`library_id`    | Id of the `library` in which the `floor` will be created in.

### Response Format

Parameter  | Description
---------  | -----------
`success`  | Whether creation was successful.
`error`    | The error message, if not successful.
`id`       | The `floor_id` of the newly created `floor` record in the database.

## get_library_list

This fetches the current list of libraries from the database. Note that no `floor` information is returned. Details of specific `floor` plans within a `library` can be retrieved via `get_library()`.

### POST Parameters

None.

### Response Format

Parameter  | Description
---------  | -----------
`success`  | Whether the retrieval was successful.
`error`    | The error message, if not successful.
`data`     | An array of Libraries without floors property. Refer to the JSON data format for more information.

## get_library

This fetches all information from a `library`, including details on its floors' geometry and aisle data.

### POST Parameters

Parameter       | Description
---------       | -----------
`library_id`    | ID of the `library` to fetch information on.

### Response Format

Parameter  | Description
---------  | -----------
`success`  | Whether creation was successful.
`error`    | The error message, if not successful.
`data`       | A `library` object. Refer to the JSON data format for more information.

## get_book_location

This takes a book's call number and its located `library` and tries to locate the exact aisle containing the book. It also provides the `floor` plan data for that `floor` so a map can be drawn from it.

If multiple floors appear to contain the same book's call number, multiple floors will be returned. It is up to the caller to decide on what to do with the data.

The book's call number is stored as a string in the database, and it is in this function that we interpret that string and compare it with the given call number. This means that this is the function to modify if we want to use another call number system or fine tune how the call numbers should be parsed.

### POST Parameters

Parameter       | Description
---------       | -----------
`library_name`  | Name of the `library` to find the book from.
`call_number`   | The book's call number in a string representation. For the format of this number in the current implementation, refer to documentation on call number.

### Response Format

Parameter  | Description
---------  | -----------
`success`  | Whether retrieval was successful.
`error`    | The error message, if not successful.
`floors`   | A numeric array of `floor` object. Refer to the JSON data format for more information.
`aisles`   | A numeric array of associative arrays each containing two elements, an 'aisle_id' and a 'side'. The former is the id of the aisle containing the given call number. The latter is present only if the aisle is double sided, in which case 'side' identifies which side the book is on. 0 indicates the book is on the left, 1 indicates the book is on the right. Left and right are relative to the local space of the stack (where we disregard its rotation). At 0 rotation, a double sided aisle should have short `width` and long height, with two rectangles: one on the left and the other on the right.

## update_library

This updates a `library`'s name. Note each `library` must have a unique name.

### POST Parameters

Parameter       | Description
---------       | -----------
`token`         | Access token given from login.
`library_id`    | Id of the `library` to perform the change to.
`library_name`  | New name of the library.

### Response Format

Parameter  | Description
---------  | -----------
`success`  | Whether the update was successful.
`error`    | The error message, if not successful.

## update_floor

This updates a `floor` of a library. The `floor` includes aisles, call number ranges in aisles, aisle areas, aisles in aisle areas, call number ranges in aisles in aisle areas, landmarks, and walls.

### POST Parameters

Parameter       | Description
---------       | -----------
`token`         | Access token given from login.
`floor`         | `floor` object encoded in JSON. Refer to the JSON data format for more information.

### Response Format

Parameter  | Description
---------  | -----------
`success`  | Whether the update was successful.
`error`    | The error message, if not successful.

## update_floor_meta

This updates a `floor`'s name and `floor` order within a library.

### POST Parameters

Parameter       | Description
---------       | -----------
`token`         | Access token given from login.
`floor_id`      | Id of the `floor` to perform the change to.
`floor_name`    | New name of the floor.
`floor_order`   | New order of the floor.

### Response Format

Parameter  | Description
---------  | -----------
`success`  | Whether the update was successful.
`error`    | The error message, if not successful.

## delete_library

This deletes a `library` and all floors associated with it. Deletions are not recoverable.

### POST Parameters

Parameter       | Description
---------       | -----------
`token`         | Access token given from login.
`library_id`    | Id of the `library` to delete.

### Response Format

Parameter  | Description
---------  | -----------
`success`  | Whether the deletion was successful.
`error`    | The error message, if not successful.

## delete_floor

This deletes a `floor` and all objects inside it. Deletions are not recoverable.

### POST Parameters

Parameter       | Description
---------       | -----------
`token`         | Access token given from login.
`floor_id`      | Id of the `floor` to delete.

### Response Format

Parameter  | Description
---------  | -----------
`success`  | Whether the deletion was successful.
`error`    | The error message, if not successful.

# Database Specifications

## library

**Description:** This table saves information of each `library` that needs to be managed, and we can relate floors, stacks, books, etc. in the library

Parameter       |  Data type  | Description
---------       |  ---------- | -----------
`library_id `   | INT <br/> NOT NULL | It is the primary key of the library, and it auto increments each time when a new `library` has been inserted into the table. It will be referred by table `floor` so that each `floor` knows which `library` it belongs to.
`library_name ` | CHAR(25) <br/> UNIQUE | It indicates the name of the library. Since it is not intuitive for administrator or the users to use id, we gives each `library` a name, so it is easier to manage, display and manipulate.

## floor

**Description:** This table saves the information for each floor, and each `floor` belongs to one and only one library. The `floor` has necessary information for generating maps and managing its relation with corresponding library.

Parameter       |  Data type  | Description
---------       |  ---------- | -----------
`floor_id` | INT <br/> NOT NULL | It is the primary key of the floor, and it auto increments each time when a new `floor` has been inserted into the table. It will be referred by table wall, aisle_area, aisle, landmark to know which `floor` they belong to.
`floor_name` | CHAR(25) | It indicates the name of the floor. Since it is not intuitive for administrator or the users to use id, we gives each `floor` a name, so it is easier to manage, display and manipulate
floor_order | FLOAT <br/> NOT NULL | It indicates the order of the `floor` in its corresponding library, so that we know which `floor` is above others and which `floor` is beneath others by comparing their floor_order. When display in may, the floor_order can be shown to user, so that users know where the `floor` locates in the library.
`library` | INT <br/> NOT NULL | It is a foreign key links to table library’s library_id, and it helps to identify which `library` the wall belongs to.

## wall

**Description:** This table saves the information of walls on the floors, and saving such information can help for map generating.

Parameter       |  Data type  | Description
---------       |  ---------- | -----------
`wall_id` | INT <br/> NOT NULL | It is the primary key of the wall, and it auto increments each time when a new wall has been inserted into the table. Even though it does not have any other use for the user, but it is a good convention to assign each wall an unique id.
`start_x` | FLOAT <br/> NOT NULL | It indicates the x-axis coordinate of where the wall starts.
`start_y` | FLOAT <br/> NOT NULL | It indicates the y-axis coordinate of where the wall starts.
`end_x` | FLOAT <br/> NOT NULL | It indicates the x-axis coordinate of where the wall ends.
`end_y` | FLOAT <br/> NOT NULL | It indicates the y-axis coordinate of where the wall ends.
`floor` | INT | It is a foreign key links to table floor’s floor_id, and it helps to identify which `floor` the wall belongs to.

## aisle_area

**Description:** This table saves information of aisle area, where aisle area is an area in a `floor` that can hold several aisles.

Parameter       |  Data type  | Description
---------       |  ---------- | -----------
`aisle_area_id` | INT <br/> NOT NULL | It is the primary key of the aisle_area, and it auto increments each time when a new `aisle_area` has been inserted into the table. It can also be referenced by table aisle, so that each aisle knows which `aisle_area` it belongs to.
`center_x` | FLOAT <br/> NOT NULL | It indicates the x-axis coordinate of the center of aisle_area.
`center_y` | FLOAT <br/> NOT NULL | It indicates the y-axis coordinate of the center of aisle_area.
`width` | FLOAT <br/> NOT NULL | It indicates the `width` of aisle_area
`height` | FLOAT <br/> NOT NULL | It indicates the `height` of aisle_area
`rotation` | FLOAT <br/> NOT NULL | It indicates angle that the aisle_area tilts relate to the floor
`floor` | INT | It is a foreign key links to table floor’s floor_id, and it helps to identify which `floor` the aisle_area belongs to.

## aisle

**Description:** This table saves information of aisle, where aisle can either has 1 or 2 sides, and it can hold books with multiple call ranges.

Parameter       |  Data type  | Description
---------       |  ---------- | -----------
`aisle_id` | INT <br/> NOT NULL | It is the primary key of the aisle. It auto increments each time when a new aisle has been inserted into the table. It can also be referenced by table call_range, so that each call_range knows which aisle it belongs to.
`center_x` | FLOAT <br/> NOT NULL | It indicates the x-axis coordinate of the center of aisle.
`center_y` | FLOAT <br/> NOT NULL | It indicates the y-axis coordinate of the center of aisle.
`width` | FLOAT <br/> NOT NULL | It indicates the `width` of aisle
`height` | FLOAT <br/> NOT NULL | It indicates the `height` of aisle
`rotation` | FLOAT <br/> NOT NULL | It indicates angle that the aisle tilts relate to the floor
is_double_sided | INT <br/> NOT NULL | It indicates if the aisle has two sides
aisle_area | INT <br/> NOT NULL | It is a foreign key links to table aisle_area’s aisle_area_id, so that the aisle knows which aisle_area it belongs to
`floor` | INT | It is a foreign key links to table floor’s floor_id, and it helps to identify which `floor` the aisle belongs to.

## call_range

**Description:** This table saves information of call_range, where each call range saves what range of books are contained, and each call_range belongs to one and only one aisle.

Parameter       |  Data type  | Description
---------       |  ---------- | -----------
call_range_id | INT <br/> NOT NULL | It is the primary key of the aisle. It auto increments each time when a new aisle has been inserted into the table
collection | CHAR(50) <br/> NOT NULL | It indicates which collection that this call_range is under.
call_start | CHAR(50) <br/> NOT NULL | It indicates the lower bound of the call_range
call_end | CHAR(50) <br/> NOT NULL | It indicates the upper bound of the call_range
side | INT | It indicates which side on the aisle that the call_range belongs to
aisle | INT <br/> NOT NULL | It is a foreign key links to table aisle’s aisle_id, and it helps to identify which aisle that the call_range locates

## landmark

**Description:** This table saves information of landmarks, which can be any non-aisle and non-aisle area objects on the `floor` that are important to note, and they can be toilets, elevators, etc.

Parameter       |  Data type  | Description
---------       |  ---------- | -----------
`landmark_id` | INT <br/> NOT NULL | It is the primary key of the landmark. It auto increments each time when a new landmark has been inserted into the table
`landmark_type` | CHAR(25) <br/> NOT NULL | It indicates what the landmark is. Currently our options includes elevators, toilets, stairs, and we can for sure have more if there is demand in the future.
`center_x` | FLOAT <br/> NOT NULL | It indicates the x-axis coordinate of the center of landmark.
`center_y` | FLOAT <br/> NOT NULL | It indicates the y-axis coordinate of the center of landmark.
`width` | FLOAT <br/> NOT NULL | It indicates the `width` of landmark
`height` | FLOAT <br/> NOT NULL | It indicates the `height` of landmark
`rotation` | FLOAT <br/> NOT NULL | It indicates angle that the landmark tilts relate to the floor
`floor` | INT <br/> NOT NULL | It is a foreign key links to table floor’s floor_id, and it helps to identify which `floor` the landmark belongs to.

## token
**Description:** This table saves the information of the tokens, which is assigned to user each time when a user login. Each token has an expiration time, and the user needs to use username and password to login again if token expires.

Parameter       |  Data type  | Description
---------       |  ---------- | -----------
`token_id`| INT <br/> NOT NULL | It is the primary key of the token. It auto increments each time when a new token has been inserted into the table
token_body | CHAR(64) | It saves the randomly generated token given to users when they login successfully with their username and password
expiration_date | DATETIME | It indicates the expiration date of a token. Once expiration_date is passed, the token can no longer be used, and therefore the user needs to login again with username and password if they wish to use the application for more time

## users

**Description:** This table saves information of usernames and passwords that can be used to login.
user_id: it is an INT, and it is the primary key of the users. It auto increments each time when a new users has been inserted into the table

Parameter       |  Data type  | Description
---------       |  ---------- | -----------
`user_id` | INT <br/> NOT NULL | It auto increments each time when a new user has been inserted into the table
`username` | CHAR(25) | It saves valid username that can be used for login
`password` | CHAR(25) | It saves corresponding password for each username that can be used for login
