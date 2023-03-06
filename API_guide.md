<img height="30" src="logo.svg" title="Keyguru logo" width="200"/>

# Keyguru API guide

API version: **2.2.0**<br>
API release date: **2022-12-01**<br>
[Swagger web interface](https://keyguru.app/api/ui/)

---
**New in version 2.2.0**

- A new endpoint "[**PATCH drawer reservation**](#patch-drawer-reservation)" was added. It allows a **full update of a
  particular reservation**.

---

## Introduction

The API server address is `https://keyguru.app/api/` supplemented by the [particular endpoint](#api-endpoints).
For example `https://keyguru.app/api/v1/self`

Keyguru API is built on [OpenAPI 3.0.0](https://swagger.io/specification/v3/). Every request must be authorized by the
API key in the `X-Api-Key` header. Please contact us at [support@keyguru.cz](mailto:support@keyguru.cz)
to get your API key.

The functionality of the API can be tested using the [Swagger web interface](https://keyguru.app/api/ui/), which also
provides a brief explanation of the individual [endpoints](#api-endpoints). This guide provides a broader context.

You can also use Swagger to automatically generate parts of your source code for integration with our API:
Download our [definition file](https://keyguru.app/api/ui/index.yaml) and paste its contents into the left column on
the [Swagger Editor page](https://editor.swagger.io/). At the top menu in the Swagger Editor is the "Generate Client"
section.

### How reservations work

**Reservation** = time window from when to when the compartment is accessible to the guest. It may copy the length of
the room reservation or be independent.

#### Mapping variants

Example: A customer has booked a room for 3 nights, expected arrival is at 2:00 in the morning.

##### 1. One room, one drawer

- The room is reserved for 3 nights, so the compartment will be reserved also for 3 nights. The customer
  can pick up the keys and return them at any time with the same access code - even repeatedly.

- **Advantages**: Simple and fully unattended.

- **Disadvantages**: This requires as many compartments as there are rooms, therefore not very efficient, especially
  during low traffic.

##### 2. Drawers on purpose

- Make a reservation for the 1st night when the customer plans to arrive. The return of the keys will either be handled
  outside the box or a new short reservation will be created for the customer.

- **Advantages:** Efficiency - this requires significantly fewer drawers than rooms.

- **Disadvantages:** Slightly more complicated, keys are returned at the reception or left at the door.

### Basic concepts

#### _X-Api-Key_ = Customer's API key

- The API requires authorization of each request using a key in the `X-Api-Key` header.

#### _Device_ = customer's access system

- Usually 1 hotel = 1 access system.
- Access system consists of individual boxes and gatekeepers which do not need to be addressed in the API requests.

#### _Drawer_ = compartment in the box

- Drawer is the smallest unit in the customer's access system.
- Reservations are made for drawers.

#### _Guest code_ = access code

- The guest access code is valid for the duration of the compartment reservation.
- Each reservation has 1 guest code.
- The guest code must be unique within the access system. Different reservations can have the same code only if
  the validity of these reservations does not overlap on the timeline.
- The guest code can be set or automatically generated.
- Staff members have other access codes regardless of reservations. Staff codes cannot be created or changed via the API
  but only in the web administration interface.

### User-defined names

_Devices_ and _drawers_ can have custom names. They can be called either by their ID or by their name.

- The _device_ name must be unique within the customer.
- The _drawer_ name must be unique within the _device_.

The reservation name **does not have** to be unique (eg. "Smith"). Therefore, the reservation can only be addressed by
its ID.

### Date and time format

Date and time values should follow the [ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html) standard.
Both UTC and time offset formats are supported:

- `2021-06-07T14:18:19Z`
- `2021-06-07T14:18:19.123Z`
- `2021-06-07T14:18:19.123456Z`
- `2021-06-07T16:18:19+02:00`

The API responds in the UTC (Z) format.

You can set a specific time zone for each _device_ in the web administration interface.

### Authorization and access control

A customer's API key allows access to all devices of that customer.

### Duplication of some GET endpoints to POST

According to [RFC 7231](https://www.rfc-editor.org/rfc/rfc7231#section-4.3.1), many HTTP clients nowadays don't
support the body in the GET requests. So we duplicated some of the GET endpoints to POST. The original GET endpoints
will be preserved for backward compatibility.

## API endpoints

### [GET self](https://keyguru.app/api/ui/#/default/get_v1_self)

`/v1/self`

This function only **validates the API key** passed in the `X-Api-Key` header and returns basic information about the
customer.

### [GET device](https://keyguru.app/api/ui/#/default/get_v1_device)

`/v1/device`

This function returns **all customer devices**.

### [GET device {identification}](https://keyguru.app/api/ui/#/default/get_v1_device__deviceIdentification_)

`/v1/device/{deviceIdentification}`

This function returns a **particular customer device**.

**Required parameters:**

- device name or ID

### [PATCH device {identification}](https://keyguru.app/api/ui/#/default/patch_v1_device__deviceIdentification_)

`/v1/device/{deviceIdentification}`

This function currently allows only the **change of the device name**. The name must be unique within all customer
devices.

**Required parameters:**

- device name or ID
- new device name

### [POST device {identification} history](https://keyguru.app/api/ui/#/default/post_v1_device__deviceIdentification__history)

`/v1/device/{deviceIdentification}/history`

This function returns the **history of the device**. All events associated with the particular device will be listed:

- events from the device (e.g. "open_drawer")
- events done by the operators (e.g. "reservation_create")
- events done by the Keyguru technical staff (e.g. "device_assign")
- events done via the API

**Required parameters:**

- device name or ID

**Optional parameters:**

- time range limit (default: all events)
- maximum number of items (default: 1000)

**NOTICE:** An event from the device gets its timestamp when it arrives on our server. The timestamp does not represent
the moment the event happened in the device. The delay is usually less than 1 minute as long as the device is online. We
will fix this in the future.

### [GET drawer](https://keyguru.app/api/ui/#/default/get_v1_device__deviceIdentification__drawer)

`/v1/device/{deviceIdentification}/drawer`

This function returns **all the drawers** in a particular customer device.

**Required parameters:**

- device name or ID

### [GET drawer {identification}](https://keyguru.app/api/ui/#/default/get_v1_device__deviceIdentification__drawer__drawerIdentification_)

`v1/device/{deviceIdentification}/drawer/{drawerIdentification}`

This function returns a **particular drawer** in the customer device.

**Required parameters:**

- device name or ID
- drawer name or ID

### [PATCH drawer {identification}](https://keyguru.app/api/ui/#/default/patch_v1_device__deviceIdentification__drawer__drawerIdentification_)

`v1/device/{deviceIdentification}/drawer/{drawerIdentification}`

This function currently allows only the **change of the drawer name**. The name must be unique within the device.

**Required parameters:**

- device name or ID
- drawer name or ID
- new drawer name

### [GET device reservation](https://keyguru.app/api/ui/#/default/get_v1_device__deviceIdentification__reservation)

`/v1/device/{deviceIdentification}/reservation`

This function returns **all reservations in the device except deleted** reservations.

**Required parameters:**

- device name or ID

This function is preserved for backward compatibility. We recommend using the newer
[**POST device reservation-list**](#post-device-reservation-list) function which also allows **filtering** and
**including deleted** reservations.

If you need only reservations **in a specific drawer**, you can use one of the following functions:

* [**POST drawer reservation-list**](#post-drawer-reservation-list)
    * **recommended**
    * allows filtering and including deleted reservations
* [**GET drawer reservation**](#get-drawer-reservation)
    * **preserved for backward compatibility**
    * does not allow filtering and including deleted reservations

### [POST device reservation](https://keyguru.app/api/ui/#/default/post_v1_device__deviceIdentification__reservation)

`/v1/device/{deviceIdentification}/reservation`

This function **creates and returns a new reservation. The drawer for the reservation will be assigned automatically.**
Reservations within 1 drawer cannot overlap. If there is no free drawer available, the function will return an error.

**Required parameters:**

- device name or ID
- reservation name
- reservation start date
- reservation end date
    - minimum duration: 1 hour

**Optional parameters:**

- guest access code
    - 5 digits
    - must be unique within all current reservations in the device
    - if not specified, the system will generate a random code
- reservation note

**NOTICE:** The access code must then be communicated to the guest, e.g. by email or SMS.

If you need to create a new reservation **in a particular drawer**, then use the
[**POST drawer reservation**](#post-drawer-reservation) function.

### [POST device reservation-list](https://keyguru.app/api/ui/#/default/post_v1_device__deviceIdentification__reservation_list)

`/v1/device/{deviceIdentification}/reservation-list`

This function returns **all reservations in the device** similar to the
[**GET device reservation**](#get-device-reservation) function with the difference that this function also allows
**filtering** and **including deleted reservations**.

**Required parameters:**

- device name or ID

**Optional parameters:**

- time range
    - from date
    - to date
- max. reservation count (default: 1000)
- include deleted reservations (default: `false`)

If you need only reservations **in a specific drawer**, you can use one of the following functions:

* [**POST drawer reservation-list**](#post-drawer-reservation-list)
    * **recommended**
    * allows filtering and including deleted reservations
* [**GET drawer reservation**](#get-drawer-reservation)
    * **preserved for backward compatibility**
    * does not allow filtering and including deleted reservations

### [GET drawer reservation](https://keyguru.app/api/ui/#/default/get_v1_device__deviceIdentification__drawer__drawerIdentification__reservation)

`/v1/device/{deviceIdentification}/drawer/{drawerIdentification}/reservation`

This function returns **all reservations in a particular drawer** in the device **except deleted reservations**.

**Required parameters:**

- device name or ID
- drawer name or ID

This function is preserved for backward compatibility. We recommend using the newer
[**POST drawer reservation-list**](#post-drawer-reservation-list) function which also allows **filtering** and
**including deleted** reservations.

If you need **all reservations from whole device**, you can use one of the following functions:

* [**POST device reservation-list**](#post-device-reservation-list)
    * **recommended**
    * allows filtering and including deleted reservations
* [**GET device reservation**](#get-device-reservation)
    * **preserved for backward compatibility**
    * does not allow filtering and including deleted reservations

### [POST drawer reservation](https://keyguru.app/api/ui/#/default/post_v1_device__deviceIdentification__drawer__drawerIdentification__reservation)

`/v1/device/{deviceIdentification}/drawer/{drawerIdentification}/reservation`

This function **creates and returns a new reservation in a particular drawer** of the device. Reservations within 1
drawer cannot overlap. If the drawer is not free, the function will return an error.

**Required parameters:**

- device name or ID
- drawer name or ID
- reservation name
- reservation start date
- reservation end date
    - minimum duration: 1 hour

**Optional parameters:**

- guest access code
    - 5 digits
    - must be unique within all current reservations in the device
    - if not specified, the system will generate a random code
- reservation note

**NOTICE:** The access code must then be communicated to the guest, e.g. by email or SMS.

If you need to **automatically find an available drawer within the device**, then use the
[**POST device reservation**](#post-device-reservation) function.

### [POST drawer reservation-list](https://keyguru.app/api/ui/#/default/post_v1_device__deviceIdentification__drawer__drawerIdentification__reservation_list)

`/v1/device/{deviceIdentification}/drawer/{drawerIdentification}/reservation-list`

This function returns **all reservations in a particular drawer** of the device similarly to the
[**GET device reservation**](#get-device-reservation) function with the difference that this function also allows
**filtering** and **including deleted reservations**.

**Required parameters:**

- device name or ID
- drawer name or ID

**Optional parameters:**

- time range
    - from date
    - to date
- max. reservation count (default: 1000)
- include deleted reservations (default: `false`)

If you need **all reservations from whole device**, you can use one of the following functions:

* [**POST device reservation-list**](#post-device-reservation-list)
    * **recommended**
    * allows filtering and including deleted reservations
* [**GET device reservation**](#get-device-reservation)
    * **preserved for backward compatibility**
    * does not allow filtering and including deleted reservations

### [PATCH drawer reservation](https://keyguru.app/api/ui/#/default/patch_v1_device__deviceIdentification__drawer__drawerIdentification__reservation__reservationId_)

`/v1/device/{deviceIdentification}/drawer/{drawerIdentification}/reservation/{reservationId}`

This function allows you to fully **change an existing reservation**. One or more attributes can be changed at once.

**Required parameters:**

- device name or ID
- drawer name or ID
- reservation ID

**Optional parameters:**

- new name of reservation
- new guest access code
    - 5 digits
    - must be unique within all current reservations in the device
    - if not specified, the code **will not be changed**
- new start date of the reservation
- new end date of the reservation
    - minimum duration: 1 hour
- new note of the reservation

### [DELETE reservation](https://keyguru.app/api/ui/#/default/delete_v1_device__deviceIdentification__drawer__drawerIdentification__reservation__reservationId_)

`/v1/device/{deviceIdentification}/drawer/{drawerIdentification}/reservation/{reservationId}`

This function **deletes an existing reservation**.

**Required parameters:**

- device name or ID
- drawer name or ID
- reservation ID

If you need to **change an existing reservation or move it to another drawer**, then use the
[**PATCH drawer reservation**](#patch-drawer-reservation) function.

### [POST new-code](https://keyguru.app/api/ui/#/default/post_v1_device__deviceIdentification__drawer__drawerIdentification__reservation__reservationId__new_code)

`/v1/device/{deviceIdentification}/drawer/{drawerIdentification}/reservation/{reservationId}/new-code`

This function **changes the guest access code** of an existing reservation.

**Required parameters:**

- device name or ID
- drawer name or ID
- reservation ID

**Optional parameters:**

- new guest access code
    - 5 digits
    - must be unique within all current reservations in the device
    - if not specified, the system will **generate a random code**
