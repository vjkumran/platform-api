API Documentation
=================

Request API Access
------------------

Drop us an email at api@resy.com and provide us with the name of your
application a technical contact email address and we'll create your
application and get you started.

Authorization
-------------

Every API request must contain the following HTTP header in order to validate
access:

    Authorization: PlatformAPI client_id="[Client ID]"

A 419 response will be generated in the event the header is missing, formatted
incorrectly or the client ID is invalid.

OAuth
-----

To allow a user to authenticate using OAuth 2.0, redirect the user to the
following URL:

    https://platform.resy.com/oauth/authenticate/?client_id=[Client ID]&redirect_uri=[Callback URL]

[Please note, you must have a trailing slash after authenticate in the URL
 above.]

Once the user has completed authentication, the URL provided in the query
string will be called with an additional parameter called "code". A
server-to-server call should then be made to exchange the code for the user's
access token.

Note, exchanging for the access token requires the use of your application
secret. The secret should never be exposed as part of an URL query string
parameter or in Javascript. To keep the secret protected, the exchange
should occur server-to-server.

When attempting to authenticate the user you can receive the following error
which will cause the user to be redirected to the specified redirect_uri
(with additional parameters passed) and indicates that your client ID is
incorrect or your application has been disabled:

    error=app_not_found
    error_message=Resy couldn't retrieve the app to log into.
    error_data={'client_id': [client ID]}

If an internal server error occurs on our side the user will be redirected
to the specified redirect_uri with additional parameters passed:

    error=code_gen_error
    error_message=There was a problem while trying to log you in.

HTTP Response Codes
-------------------

### 400

    A bad request was made because of a data validation error.  Either a
    required parameter is missing or failed a custom software validation
    check.

    {
        "message": "[text]"
    }

    or

    {
        "status": 400,
        "message": Invalid data received.",
        "data": {
            "[field name 1]": "[validation failure 1]",
            "[field name 2]": "[validation failure 2]",
            ...
            "[field name N]": "[validation failure N]"
        }
    }

### 402

    The user has not provided payment methods.

    {
        "status": 402,
        "message": "Payment Required"
    }

### 403

    The request is forbidden most likely because something has gone wrong.

    {
        "status": 403,
        "message": "[message]"
    }

### 404

    The object being requested could not be found.

    {
        "status": 404,
        "message": "Not Found"
    }

### 409

    The user's account does not exist or has been deactivated.  The user's
    Resy data should be wiped from the requesting device and the user should be
    logged out.  If you are storing an access_token for the user it should be
    considered no longer valid and purged.

    {
        "status": 409,
        "message": "Conflict"
    }

### 412

    A pre-condition for completing the request failed.  The response data will
    be customized for each condition and documented where appropriate.

### 419

    The user could not be authenticated.

    {
        "status": 419,
        "message": "Authentication Timeout"
    }

A Note About the resy_token
---------------------------

The resy_token values are expired on our servers and will change from request
to request.  The data stored within them may also change with no notification
to you.  As a result you should never store these tokens.

For the sake of clarity, the access_token values never change and can be
stored forever.

Application Payment Type
------------------------

Your application can select one of two payment modes:

    Concierge
    User

In the case of concierge, you provide Resy with a credit card for charges.  In
user mode, the user is presented with a credit card input form during the
OAuth 2.0 flow and that information is used for all charges.

End Points
----------

# /1/oauth/token

### GET

    Exchanges a code for a valid access token.

    +-------------------------------------------------------------------------+
    | Parameter Name    | Req (Y/N) | Details                                 |
    |-------------------|-----------|-----------------------------------------|
    | secret            |     Y     |                                         |
    |-------------------|-----------|-----------------------------------------|
    | code              |     Y     |                                         |
    +-------------------------------------------------------------------------+

    +-------------------------------------------------------------------------+
    | Response Code | Details                                                 |
    |---------------|---------------------------------------------------------|
    | 200           |                                                         |
    |---------------|---------------------------------------------------------|
    | 400           |                                                         |
    |---------------|---------------------------------------------------------|
    | 419           |                                                         |
    +-------------------------------------------------------------------------+

    +-------------------------------------------------------------------------+
    | Response                                                                |
    +-------------------------------------------------------------------------+
    | {                                                                       |
    |   "access_token": "abcdefghijklmnopqrstuvwxyz1234567890abcdefghijkl..." |
    | }                                                                       |
    +-------------------------------------------------------------------------+

# /1/reservation

### DELETE

    Cancels a reservation.

    +-------------------------------------------------------------------------+
    | Parameter Name    | Req (Y/N) | Details                                 |
    |-------------------|-----------|-----------------------------------------|
    | access_token      |     Y     |                                         |
    |-------------------|-----------|-----------------------------------------|
    | resy_token        |     Y     |                                         |
    +-------------------------------------------------------------------------+

    +-------------------------------------------------------------------------+
    | Response Code | Details                                                 |
    |---------------|---------------------------------------------------------|
    | 200           |                                                         |
    |---------------|---------------------------------------------------------|
    | 400           |                                                         |
    |---------------|---------------------------------------------------------|
    | 403           | Attempting to execute a payment transaction failed      |
    |               | because of an error from the payment processor.  The    |
    |               | message included with this error is human readable and  |
    |               | explains why the transaction failed.                    |
    |---------------|---------------------------------------------------------|
    | 412           | The cancellation policy for the reservation prevents    |
    |               | it from being cancelled.                                |
    |---------------|---------------------------------------------------------|
    | 419           |                                                         |
    +-------------------------------------------------------------------------+

    +-------------------------------------------------------------------------+
    | Response                                                                |
    +-------------------------------------------------------------------------+
    | None                                                                    |
    +-------------------------------------------------------------------------+

### GET

    Gets the details for a specific available reservation.

    +-------------------------------------------------------------------------+
    | Parameter Name    | Req (Y/N) | Details                                 |
    |-------------------|-----------|-----------------------------------------|
    | id                |     Y     |                                         |
    |-------------------|-----------|-----------------------------------------|
    | day               |     Y     |                                         |
    |-------------------|-----------|-----------------------------------------|
    | num_seats         |     Y     |                                         |
    +-------------------------------------------------------------------------+

    +-------------------------------------------------------------------------+
    | Response Code | Details                                                 |
    |---------------|---------------------------------------------------------|
    | 200           |                                                         |
    |---------------|---------------------------------------------------------|
    | 400           |                                                         |
    |---------------|---------------------------------------------------------|
    | 404           | Given the parameters, the reservation could not be      |
    |               | found. This can occur if, for example, an ID is         |
    |               | provided for a reservation that has a maximum party     |
    |               | size of 2 but num_seats is specified as 4.              |
    |---------------|---------------------------------------------------------|
    | 419           |                                                         |
    +-------------------------------------------------------------------------+

    +-------------------------------------------------------------------------+
    | Response                                                                |
    +-------------------------------------------------------------------------+
    | {                                                                       |
    |   "cancellation_policy": [                                              |
    |       "This reservation can be changed until Jan 1, 2015.",             |
    |       "This reservation cannot be cancelled."                           |
    |   ],                                                                    |
    |   "double_confirmation": [                                              |
    |       "Are you sure you want to book this reservation?"                 |
    |   ],                                                                    |
    |   "payment": {                                                          |
    |       "balance": {                                                      |
    |           "modifier": "Credit Applied",                                 |
    |           "value": "$30"                                                |
    |       },                                                                |
    |       "buy": {                                                          |
    |           "action": "RESERVE NOW",                                      |
    |           "after_modifier": "",                                         |
    |           "before_modifier": "",                                        |
    |           "value": "$100 x 2 = $200"                                    |
    |       },                                                                |
    |       "description": [                                                  |
    |           "$80 per person",                                             |
    |           "$15 service charge",                                         |
    |           "$5 taxes and fees"                                           |
    |       ]                                                                 |
    |   },                                                                    |
    |   "reservation": {                                                      |
    |       "day": "2015-01-01",                                              |
    |       "deep_link": "resy://resy.com/ReservationDetails?venue_id=1&...", |
    |       "features": [                                                     |
    |           "Test Venue donates 100% of the proceeds from this table t.." |
    |       ],                                                                |
    |       "menu_items": [                                                   |
    |           "Delicious Vegan Burger",                                     |
    |           "Nom nom nom, brussel sprouts!"                               |
    |       ],                                                                |
    |       "num_seats": 2,                                                   |
    |       "seat_type": "Communal",                                          |
    |       "time_slot": "19:30:00"                                           |
    |   },                                                                    |
    |   "resy_token": "WTYxbM_YsLOcA1cTRZTOp_HHQh5ejxfXq7gUZ4gjqcme8mnbs...", |
    |   "venue": {                                                            |
    |       "about": "Are you ready to eat vegan? Well you better be!",       |
    |       "contact": {                                                      |
    |           "phone_number": "2125551212",                                 |
    |           "url": "http://resy.com/"                                     |
    |       },                                                                |
    |       "deep_link": "resy://resy.com/VenueDetails?venue_id=1&day=20...", |
    |       "images": [                                                       |
    |           "'https://s3.amazonaws.com/resy.com/images/venues/1/1.jpg"    |
    |       ],                                                                |
    |       "location": {                                                     |
    |           "address_1": "315 Park Avenue",                               |
    |           "address_2": null,                                            |
    |           "city": "New York",                                           |
    |           "cross_street_1": "23rd Street",                              |
    |           "cross_street_2": "24th Street",                              |
    |           "latitude": 40.745812,                                        |
    |           "longitude": -73.9822091,                                     |
    |           "neighborhood": "Flatiron",                                   |
    |           "postal_code": "10016",                                       |
    |           "state": "NY",                                                |
    |           "time_zone": "EST5EDT"                                        |
    |       },                                                                |
    |       "name": "Test Venue",                                             |
    |       "price_range_id": 4,                                              |
    |       "rater": {                                                        |
    |           "image": "https://s3.amazonaws.com/resy.com/images/rater...", |
    |           "name": "Resy",                                               |
    |           "scale": 5,                                                   |
    |           "score": 5.0                                                  |
    |       },                                                                |
    |       "tagline": "Yummy food!",                                         |
    |       "travel_time": {                                                  |
    |           "distance": 0.0,                                              |
    |           "driving": 1,                                                 |
    |           "walking": 1                                                  |
    |       },                                                                |
    |       "type": "Vegan Joint"                                             |
    |   }                                                                     |
    | }                                                                       |
    +-------------------------------------------------------------------------+

    +-------------------------------------------------------------------------+
    | Notes                                                                   |
    +-------------------------------------------------------------------------+
    | - If the "double_confirmation" array is not empty, you must display the |
    |   values in it (in the order they are provided) and enforce that the    |
    |   user agree to them as a separate action prior to booking the          |
    |   reservation.  For example, you may present the user with a check box  |
    |   that they must check as confirmation or you might fire an overlay     |
    |   when the user taps book that requires a second tap after presenting   |
    |   the values.                                                           |
    +-------------------------------------------------------------------------+

### POST

    Books a reservation for a user.

    +-------------------------------------------------------------------------+
    | Parameter Name    | Req (Y/N) | Details                                 |
    |-------------------|-----------|-----------------------------------------|
    | access_token      |     Y     |                                         |
    |-------------------|-----------|-----------------------------------------|
    | resy_token        |     Y     |                                         |
    +-------------------------------------------------------------------------+

    +-------------------------------------------------------------------------+
    | Response Code | Details                                                 |
    |---------------|---------------------------------------------------------|
    | 201           |                                                         |
    |---------------|---------------------------------------------------------|
    | 400           |                                                         |
    |---------------|---------------------------------------------------------|
    | 402           | The user does not have a payment method on file.        |
    |---------------|---------------------------------------------------------|
    | 403           | Attempting to execute a payment transaction failed      |
    |               | because of an error from the payment processor.  The    |
    |               | message included with this error is human readable and  |
    |               | explains why the transaction failed.                    |
    |---------------|---------------------------------------------------------|
    | 404           | The reservation has expired.                            |
    |---------------|---------------------------------------------------------|
    | 412           | The user already has a reservation that conflicts with  |
    |               | the one they are attempting to book.  They cannot       |
    |               | book the new one. The response data will be formatted   |
    |               | as follows representing the existing reservation:       |
    |               |                                                         |
    |               | {                                                       |
    |               |   "specs": {                                            |
    |               |       "day": "2015-01-01",                              |
    |               |       "time_slot": "19:30:00"                           |
    |               |   }                                                     |
    |               |   "venue": {                                            |
    |               |       "id": 1,                                          |
    |               |       "name": "Test Venue"                              |
    |               |   }                                                     |
    |               | }                                                       |
    |---------------|---------------------------------------------------------|
    | 419           |                                                         |
    +-------------------------------------------------------------------------+

    +-------------------------------------------------------------------------+
    | Response                                                                |
    +-------------------------------------------------------------------------+
    | {                                                                       |
    |   "resy_token": "WTYxbM_YsLOcA1cTRZTOp_HHQh5ejxfXq7gUZ4gjqcme8mnbsr..." |
    | }                                                                       |
    +-------------------------------------------------------------------------+

# /1/reservation/find

### GET

    Gets all of the available reservations based on the search criteria.

    +-------------------------------------------------------------------------+
    | Parameter Name    | Req (Y/N) | Details                                 |
    |-------------------|-----------|-----------------------------------------|
    | long              |     Y     |                                         |
    |-------------------|-----------|-----------------------------------------|
    | lat               |     Y     |                                         |
    |-------------------|-----------|-----------------------------------------|
    | day               |     Y     |                                         |
    |-------------------|-----------|-----------------------------------------|
    | num_seats         |     Y     |                                         |
    +-------------------------------------------------------------------------+

    +-------------------------------------------------------------------------+
    | Response Code | Details                                                 |
    |---------------|---------------------------------------------------------|
    | 200           |                                                         |
    |---------------|---------------------------------------------------------|
    | 400           |                                                         |
    |---------------|---------------------------------------------------------|
    | 419           |                                                         |
    +-------------------------------------------------------------------------+

    +-------------------------------------------------------------------------+
    | Response                                                                |
    +-------------------------------------------------------------------------+
    | {                                                                       |
    |   "available": [                                                        |
    |       {                                                                 |
    |           "deep_link": "resy://resy.com/VenueDetails?venue_id=1",       |
    |           "images": [                                                   |
    |               "https://s3.amazonaws.com/resy.com/images/venue/1/1.jpg"  |
    |           ],                                                            |
    |           "location": {                                                 |
    |               "city": "New York",                                       |
    |               "latitude": 40.745812,                                    |
    |               "longitude": -73.9822091,                                 |
    |               "neighborhood": "Flatiron",                               |
    |               "time_zone": "EST5EDT"                                    |
    |           },                                                            |
    |           "name": "Test Venue",                                         |
    |           "price_range_id": 4,                                          |
    |           "reservations": [                                             |
    |               {                                                         |
    |                   "deep_link": "resy://resy.com/ReservationDetails...", |
    |                   "id": 1,                                              |
    |                   "seat_type": "Communal",                              |
    |                   "time_slot": "19:30:00"                               |
    |               }                                                         |
    |           ],                                                            |
    |           "travel_time": {                                              |
    |               "distance": 0.005240127936334476,                         |
    |               "driving": 1,                                             |
    |               "walking": 1                                              |
    |           },                                                            |
    |           "type": "Vegan"                                               |
    |       }                                                                 |
    |   ]                                                                     |
    | }                                                                       |
    +-------------------------------------------------------------------------+

# /1/reservation/find/[location]

### GET

    Gets all of the available reservations based on the search criteria in a
    specific Resy market.

    +-------------------------------------------------------------------------+
    | Parameter Name    | Req (Y/N) | Details                                 |
    |-------------------|-----------|-----------------------------------------|
    | day               |     Y     |                                         |
    +-------------------------------------------------------------------------+

    +-------------------------------------------------------------------------+
    | Response Code | Details                                                 |
    |---------------|---------------------------------------------------------|
    | 200           |                                                         |
    |---------------|---------------------------------------------------------|
    | 400           |                                                         |
    |---------------|---------------------------------------------------------|
    | 404           | The location value provided does not exist as a Resy    |
    |               | market.                                                 |
    |---------------|---------------------------------------------------------|
    | 419           |                                                         |
    +-------------------------------------------------------------------------+

    +-------------------------------------------------------------------------+
    | Response                                                                |
    +-------------------------------------------------------------------------+
    | {                                                                       |
    |   "available": [                                                        |
    |       {                                                                 |
    |           "deep_link": "resy://resy.com/VenueDetails?venue_id=1",       |
    |           "images": [                                                   |
    |               "https://s3.amazonaws.com/resy.com/images/venue/1/1.jpg"  |
    |           ],                                                            |
    |           "location": {                                                 |
    |               "city": "New York",                                       |
    |               "latitude": 40.745812,                                    |
    |               "longitude": -73.9822091,                                 |
    |               "neighborhood": "Flatiron",                               |
    |               "time_zone": "EST5EDT"                                    |
    |           },                                                            |
    |           "name": "Test Venue",                                         |
    |           "price_range_id": 4,                                          |
    |           "reservations": [                                             |
    |               {                                                         |
    |                   "deep_link": "resy://resy.com/ReservationDetails...", |
    |                   "id": 1,                                              |
    |                   "min_seats": 2,                                       |
    |                   "max_seats": 4,                                       |
    |                   "seat_type": "Communal",                              |
    |                   "time_slot": "19:30:00"                               |
    |               }                                                         |
    |           ],                                                            |
    |           "travel_time": {                                              |
    |               "distance": 0.005240127936334476,                         |
    |               "driving": 1,                                             |
    |               "walking": 1                                              |
    |           },                                                            |
    |           "type": "Vegan"                                               |
    |       }                                                                 |
    |   ]                                                                     |
    | }                                                                       |
    +-------------------------------------------------------------------------+

# /1/user/reservations

### GET

    Gets all of the reservations booked by a user.

    +-------------------------------------------------------------------------+
    | Parameter Name    | Req (Y/N) | Details                                 |
    |-------------------|-----------|-----------------------------------------|
    | access_token      |     Y     |                                         |
    +-------------------------------------------------------------------------+

    +-------------------------------------------------------------------------+
    | Response Code | Details                                                 |
    |---------------|---------------------------------------------------------|
    | 200           |                                                         |
    |---------------|---------------------------------------------------------|
    | 400           |                                                         |
    |---------------|---------------------------------------------------------|
    | 419           |                                                         |
    +-------------------------------------------------------------------------+

    +-------------------------------------------------------------------------+
    | Response                                                                |
    +-------------------------------------------------------------------------+
    | {                                                                       |
    |   "reservations": [                                                     |
    |       {                                                                 |
    |           "cancellation_policy": [                                      |
    |               "This reservation can be changed until Jan 1, 2015.",     |
    |               "This reservation cannot be cancelled."                   |
    |           ],                                                            |
    |           "reservation": {                                              |
    |               "cancellation": {                                         |
    |                   "allowed": true,                                      |
    |                   "date_credit_cut_off": null,                          |
    |                   "date_refund_cut_off": "2014-12-14T12:34:56Z"         |
    |               },                                                        |
    |               "change": {                                               |
    |                   "allowed": true,                                      |
    |                   "date_cut_off": "2014-12-14T12:34:56Z"                |
    |               },                                                        |
    |               "day": "2015-01-01",                                      |
    |               "features": [                                             |
    |                   "Test Venue donates 100% of the proceeds from this.." |
    |               ],                                                        |
    |               "num_seats": 2,                                           |
    |               "seat_type": "Communal",                                  |
    |               "time_slot": "19:30:00"                                   |
    |           },                                                            |
    |           "resy_token": "WTYxbM_YsLOcA1cTRZTOp_HHQh5ejxfXq7gUZ4gjq...", |
    |           "venue": {                                                    |
    |               "images": [                                               |
    |                   "https://s3.amazonaws.com/resy.com/images/venue/i..." |
    |               ],                                                        |
    |               "location": {                                             |
    |                   "address_1": "315 Park Avenue",                       |
    |                   "address_2": null,                                    |
    |                   "city": "New York",                                   |
    |                   "cross_street_1": "23rd Street",                      |
    |                   "cross_street_2": "24th Street",                      |
    |                   "latitude": 40.745812,                                |
    |                   "longitude": -73.9822091,                             |
    |                   "neighborhood": "Flatiron",                           |
    |                   "postal_code": "10016",                               |
    |                   "state": "NY",                                        |
    |                   "time_zone": "EST5EDT"                                |
    |               },                                                        |
    |               "name": "Test Venue",                                     |
    |               "type": "Vegan"                                           |
    |           }                                                             |
    |       }                                                                 |
    |   ]                                                                     |
    | }                                                                       |
    +-------------------------------------------------------------------------+

# /1/venue/find

### GET

    Gets the reservations available for a specific venue.

    +-------------------------------------------------------------------------+
    | Parameter Name    | Req (Y/N) | Details                                 |
    |-------------------|-----------|-----------------------------------------|
    | id                |     Y     | The Foursquare ID or Google Places ID   |
    |                   |           | based on the value set for provider.    |
    |-------------------|-----------|-----------------------------------------|
    | provider          |     Y     | Available values are "foursquare" or    |
    |                   |           | "google".                               |
    |-------------------|-----------|-----------------------------------------|
    | day               |     Y     |                                         |
    |-------------------|-----------|-----------------------------------------|
    | num_seats         |     Y     |                                         |
    +-------------------------------------------------------------------------+

    +-------------------------------------------------------------------------+
    | Response Code | Details                                                 |
    |---------------|---------------------------------------------------------|
    | 200           |                                                         |
    |---------------|---------------------------------------------------------|
    | 400           |                                                         |
    |---------------|---------------------------------------------------------|
    | 404           |                                                         |
    |---------------|---------------------------------------------------------|
    | 419           |                                                         |
    +-------------------------------------------------------------------------+

    +-------------------------------------------------------------------------+
    | Response                                                                |
    +-------------------------------------------------------------------------+
    | {                                                                       |
    |   "available": [                                                        |
    |       {                                                                 |
    |           "deep_link": "resy://resy.com/VenueDetails?venue_id=1",       |
    |           "images": [                                                   |
    |               "https://s3.amazonaws.com/resy.com/images/venue/1/1.jpg"  |
    |           ],                                                            |
    |           "location": {                                                 |
    |               "city": "New York",                                       |
    |               "latitude": 40.745812,                                    |
    |               "longitude": -73.9822091,                                 |
    |               "neighborhood": "Flatiron",                               |
    |               "time_zone": "EST5EDT"                                    |
    |           },                                                            |
    |           "name": "Test Venue",                                         |
    |           "price_range_id": 4,                                          |
    |           "reservations": [                                             |
    |               {                                                         |
    |                   "deep_link": "resy://resy.com/ReservationDetails...", |
    |                   "id": 1,                                              |
    |                   "seat_type": "Communal",                              |
    |                   "time_slot": "19:30:00"                               |
    |               }                                                         |
    |           ],                                                            |
    |           "type": "Vegan"                                               |
    |       }                                                                 |
    |   ]                                                                     |
    | }                                                                       |
    +-------------------------------------------------------------------------+

# /1/venue

### GET

    Gets details about a particular venue based on the parameters.

    +-------------------------------------------------------------------------+
    | Parameter Name    | Req (Y/N) | Details                                 |
    |-------------------|-----------|-----------------------------------------|
    | id                |     Y     | The Foursquare ID or Google Places ID   |
    |                   |           | based on the value set for provider.    |
    |-------------------|-----------|-----------------------------------------|
    | provider          |     Y     | Available values are "foursquare" or    |
    |                   |           | "google".                               |
    +-------------------------------------------------------------------------+

    +-------------------------------------------------------------------------+
    | Response Code | Details                                                 |
    |---------------|---------------------------------------------------------|
    | 200           |                                                         |
    |---------------|---------------------------------------------------------|
    | 400           |                                                         |
    |---------------|---------------------------------------------------------|
    | 404           |                                                         |
    |---------------|---------------------------------------------------------|
    | 419           |                                                         |
    +-------------------------------------------------------------------------+

    +-------------------------------------------------------------------------+
    | Response                                                                |
    +-------------------------------------------------------------------------+
    | {                                                                       |
    |   "deep_link": "resy://resy.com/VenueDetails?venue_id=1",               |
    |   "id": 1,                                                              |
    |   "images": [                                                           |
    |       "https://s3.amazonaws.com/resy.com/images/venue/1/1.jpg"          |
    |   ],                                                                    |
    |   "location": {                                                         |
    |       "city": "New York",                                               |
    |       "latitude": 40.745812,                                            |
    |       "longitude": -73.9822091,                                         |
    |       "neighborhood": "Flatiron",                                       |
    |       "time_zone": "EST5EDT"                                            |
    |   },                                                                    |
    |   "name": "Test Venue",                                                 |
    |   "price_range_id": 4,                                                  |
    |   "type": "Vegan"                                                       |
    | }                                                                       |
    +-------------------------------------------------------------------------+
