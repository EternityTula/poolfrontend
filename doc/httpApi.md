# User management

## usercreate
Register new user and send email with activation link if parameter <poolfrontend.smtpEnabled> is "true" in pool config. Activation link format is http://<poolfrontend.poolHostAddress><poolfrontend.poolActivateLinkPrefix><actionId>, where poolfrontend.poolHostAddress and poolfrontend.poolActivateLinkPrefix defined in pool config; actionId: 512-bit unique identifier of operation, that can be user as input parameter of 'useraction' api.
Pool frontend must have a handler for configured activation link fornat.

### query type: POST
### arguments:
* login: Unique user identifier (up to 64 characters)
* password: Password (8-64 characters length)
* email: User email
* name: User name (display instead of login in 'blocks found by pool' table (up to 256 characters)

### return values:
* status: can be "ok" or one of defined error codes:
  * login_format_invalid
  * password_format_invalid
  * email_format_invalid
  * name_format_invalid
  * duplicate_email: already have another user with requested email
  * duplicate_login: already have user with requested login
  * smtp_client_create_error: internal error with SMTP protocol client
  * email_send_error: error received from SMTP server, details in pool log

### curl example:
```
curl -X POST -d "{\"login\": \"user\", \"password\": \"12345678\", \"email\": \"my@email.com\"}" http://localhost:18880/api/usercreate
```

### response examples:
```
{"status": "ok"}
```
```
{"status": "duplicate_email"}
```

### activation link format example:
```
http://localhost/actions/useracivate?id=6310abb30747f6498a5ec114fdfcc844babdbd9566bcc69e9a2472536a6a850892f339e0866215140497e186710cc15af5582de5e222e4e4a6089dcfd0270017
```

## useraction
Confirm various user actions, such as user activation, changing password, etc

### query type: POST
### arguments:
* id: unique identifier of operation generated by another api function

### return values:
* status: can be "ok" or one of defined error codes:
  * unknown_id: invalid or already activated id
  * unknown_login: internal error
  * user_already_active: internal error
  * unknown_type: internal error

### curl example:
```
curl -X POST -d "{\"id\": \"6310abb30747f6498a5ec114fdfcc844babdbd9566bcc69e9a2472536a6a850892f339e0866215140497e186710cc15af5582de5e222e4e4a6089dcfd0270017\"}" http://localhost:18880/api/useraction
```
### responses examples:
```
{"status": "ok"}
```
```
{"status": "unknown_id"}
```

## userlogin
Log in procedure, function accepted login/password and returns session id unique for user

### query type: POST
### arguments:
* login
* password

### return values:
* sessionid: unique session identifier, needed for other api functions
* status: can be "ok" or one of defined error codes:
  * unknown_login: login does not exists in pool database (this error code can be removed, not safe)
  * invalid_password: login/password mismatch
  * user_not_active: user registered, but not activated using special link sent to email

### curl example:
```
curl -X POST -d "{\"login\": \"user\", \"password\": \"12345678\"}" http://localhost:18880/api/userlogin
```
### responses examples:
```
{"sessionid": "863fe99ef908bc4ba7e954c381224f0370d8840ef6c653b14eba865caafb87c4aa2635312099a72aedc450c8dfa2d87e37641271d927c474b661afc73552d9fc","status": "ok"}
```
```
{"sessionid": "","status": "user_not_active"}
```
```
{"sessionId": "","status": "invalid_password"}
```

## userlogout
Close user session, invalidate session id

### query type: POST
### arguments:
* id: unique user session id returned by userlogin function

### return values:
* status: can be "ok" or one of defined error codes:
  * unknown_id: invalid session id
  * unknown_login: session id refers to non-existent user (possible corrupted database)

### curl example:
```
curl -X POST -d "{\"id\": \"d6c6a5b8839f4af4eac0e085a25d87efa27c56be6930763877a1410238d6d16b8e83f080719d7d0e6a0a7f927b257f39328ec67922dbdbf5a31c09d9e9413071\"}" http://localhost:18880/api/userlogout
```

### responses examples:
```
{"status": "ok"}
```
```
{"status": "unknown_id"}
```