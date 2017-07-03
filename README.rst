.. image:: https://badge.fury.io/py/smsgateway-gammu.svg
    :target: https://badge.fury.io/py/smsgateway-gammu

================
smsgateway-gammu
================

Send SMS messages via REST api using Gammu as backend.

Requirements
------------

* The following program works both on Linux and OSX.

* You must already have `Gammu <https://wammu.eu/docs/manual/project>`_ installed and configured on the right device.
Note it's obvious that your device must manage SMS sending/receiving with Gammu perfectly.

Installation
------------

* Install the package into your project's virtual environment :

  ``pip install smsgateway-gammu``

Configuration
-------------

* Modify the ``/etc/smsgateway.yml`` configuration file with your favorite text editor.

.. code-block:: yaml

  # Server settings
  server:
  # Listen address, put 0.0.0.0 to listen on all interfaces
  host: "127.0.0.1"

  # Listen port
  port: 1234

  # Gammu settings
  gammu:
    # PIN code, enter pin code there to unlock directly from application
    pin: 1234

    # Configuration : instead of a .gammurc file you can give configuration there
    # config:
    #   Device: /dev/tty.HUAWEIMobile-Modem
    #   Connection: at

  # General settings
  # Phone numbers that doesn't want to receive messages from gateway
  send_blacklist: ["+33689898989", "+33652525252"]

  # Phone numbers that are allowed to send messages to gateway
  receive_whitelist: ["+33638383838", "+33614141414"]

Usage
-----

Start the gateway
~~~~~~~~~~~~~~~~~

* To start the gateway, just type the command :

  ``smsgateway``

* Or if configuration file is elsewhere than ``/etc/smsgateway.yml`` :

  ``smsgateway --config <your_configuration_file>``

Sending a text message
~~~~~~~~~~~~~~~~~~~~~~

There are different ways to send a message :

**GET method**

``GET /sms?number=mynumbers&message=mymessage``

Where **mynumbers** are phone numbers, comma separated.

For instance : ``GET /sms?number=+33565656565,+33689898989&message=Hello``

Will send "Hello" message to the 2 phone numbers and return the JSON : ``{"status": "ok", "message": "messages sent"}``.

**POST method**

::

  POST /sms?number=mynumbers
  {
    "message": "Hello World"
  }

Where POST data is JSON containing the field string "message".

::

  POST /sms?number=mynumbers
  {
    "messages": [
      "Hello",
      "World"
    ]
  }

Where JSON contains the field "messages" which is a list for multiple messages.

User interaction
~~~~~~~~~~~~~~~~

Users allowed to send SMS messages to the gateway (receive_whitelist in config) can interact with the system.

**Keywords are :**

* **PING** : to check if the gateway works correctly, returns PONG if all is good.
* **PAUSE** : temporary pause messages coming from the gateway. It's worth to have this feature when a monitoring system spams you. To enable again just resend **PAUSE**.

Prometheus Alertmanager
~~~~~~~~~~~~~~~~~~~~~~~

The gateway was primary designed to work with Prometheus `Alertmanager <https://prometheus.io/docs/alerting/alertmanager/>`_, allowing sending alerts with SMS messages.

To configure it, create a new **receiver** entry in the alertmanager configuration file :

.. code-block:: yaml

  receivers:
    ...

    ...
    - name: sms
      webhook_configs:
        - url: http://localhost:9876/sms?number=+33623232323,+33690909090
