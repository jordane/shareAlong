===============================================================
The Official OSU Linux Users Group Decentralized Library Webapp
===============================================================

Overview
--------
This application will enable anybody to launch and host a decentralized lending
service. Specifically tailored for books but also for media, tools, and any
object which can be uniquely identified. IT HAS TO BE A PHYSICAL THING.

Ideally this will be a simple CRUD (nonono don't delete the books), API driven, webapp so it can be extended fairly easily.

One design to keep in mind is that this should not be a project *just* for lug
but rather a project by lug for the FOSS community at large. This means it
should be well documented, and not impossible for a community memeber outside
of LUG and LUG support to get started with it and host their own instance of
the project.

This document is not intended to propose implementation although considering
many of the members are pythonistas an implementation with Django would not be
unheard of.

An alternate API design
-----------------------
The api is based on resources. Urls are of one of these forms:
-------- /api/:version/:resource/
- /api/:version/:resource/:primaryKey/
- /api/:version/:resource/:action/
- /api/:version/:resource/:primaryKey/:action


Action end points are for things that aren't normal CRUD actions (if needed).
Each endpoint may have permissions required.

All objects implicitly have a automatic database-incrementing ID. Many objects
use something else for the primary key though. Primary keys are used in urls

Generally, to create a new resource:

.. code::

    POST /api/1/thing/ {
        "name": "100 Years of Solitude",
        "description": "A fantasy book about latin american politics (spoilers?)",
        ...
    }

That responds with `201 CREATED` and the contents of the object created,
including the URL of the resource

.. code::

    {
      "name": "100 Years of Solitude",
      "description": "A fantasy book about latin american politics (spoilers?)",
      "id": 42,
      "url": "http://stuffiverse.example.com/api/1/thing/42/",
      ...
    }

To modify a resource:

.. code::

    PUT /api/1/thing/42/ {
        "description": "If I told you, it would be a spoiler",
    }

Partial updates are allowed. The response is the full object description, just
like when the object was created.

To view a resource, `GET /api/1/thing/42/`. The responds the same as PUT or POST
does.

To view many resouce, `GET /api/1/thing/`. This returns a list of all matching
resources. It may be filtered like `/api/1/thing/?current_owner=mythmon`


Resource Types
- User (/api/:version/user/[username]/
    - primary id: username
    - properties
        - username (read-only)
        - password (write-only, only by permission)
        - email (read public, write by permission)
        - bio (free form text. read public, write by permission)

- Thing
    - properties
        - name
        - description
        - type (FK to ThingType)
        - current owner (FK to user)
        - original owner (FK to user)
        - datetime added to library
        - last modified datetime
        - free form key/value. (JSONField?)

- Type (/api/:version/type/[slug]/)
    - primary id: username
    - properties
        - name
        - slug

- TransferRequest
    - Anyone can create a TransferRequest. It has an object, and the potential
      new owner 
    - only the current possesor of the object can approve the transfer request.
    - workflow:
        - Bob wants to borrow a bug Alice has (which already exists in the
          system)
        - Bob creates a transfer request. Alice is notified.  ( POST
          /api/transferRequest/ {thing: 723, newOwner: 'bob'} )
        - Alice gives Bob the book in meatspace.
        - Alice marks the transferrequest as completed  ( POST
          /api/transferRequest/723/approve/ )  # This is an extra action,
          because it has other effects like modifying a Thing
            - or Alice doesn't want to give up, so she denies it ( POST
              /api/transferRequest/723/deny/ )
    - properties
        - thing (FK to Thing)
        - new owner (FK to user)
        - creation datetime
        - last modifed date time
        - state (not directly writable, only via special actions)
            - pending
            - denied
            - completed


Permissions
-----------
Most actions require permissions. These are not tied to a particular kind of
user, but are things like "Allowed to delete things". These permissions can
then be given to groups of users.

For example, super users have all permissions automatically (this is a property
of Django). Trusted users can add things. Anyone can create a transfer request,
but only an admin or the current possesor of the object being transferred can
approve the transfer.  The original creator of the transfer can cancel the
transaction.

.. Note:: This API is *strongly* influenced by the design of the library Django
    Rest Framework, and I highly suggest that library be used to implement this
    pattern. It really helps
