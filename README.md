# About
LLDAP-CLI is a command line interface for [LLDAP](https://github.com/lldap/lldap).

LLDAP uses [GraphQL](https://en.wikipedia.org/wiki/GraphQL) to offer an HTTP-based API. This API is used by an included web-based user interface. Unfortunately, [LLDAP lacks a command-line interface](https://github.com/lldap/lldap/issues/707), which is a necessity for any serious administrator. LLDAP-CLI translates CLI commands to GraphQL API calls.

# Features
- All functionality of LLDAP's web interface is supported by LLDAP-CLI.
  - User management. List, create and delete users and get and set their attributes. Also manage group membership. Users can be referred to based on usernames or email addresses (which are both unique in LLDAP).
  - Group management. List, create and delete groups and get and update their attributes.
  - LDAP schema management. Create, delete and use custom user and group attributes and object classes. Use the latest development version of LLDAP to fully support this feature.
- Different authentication options: environment variables, command line options, and retrieving credentials from LLDAP's configuration file. LLDAP-CLI can use a username and password to authenticate, or JSON web tokens.
- Different session options. LLDAP-CLI can use a username and password to get JSON web tokens. These tokens can be used for a single command, after which LLDAP-CLI will logout the session immediately (default behavior). Alternatively, the tokens can be re-used for multiple commands, reducing the system load and time needed for bulk operations.
- Portable. LLDAP-CLI is written in Bash. Aside from LLDAP, its dependencies should be available on most systems either directly or as installable packages from official repositories.

# Requirements
- [LLDAP](https://github.com/lldap/lldap). This tool will be a bit useless if LLDAP is not available. Furthermore, `lldap_set_password` is neccessary to set user passwords.
- [GNU Bash](https://www.gnu.org/software/bash/). Some features specific to Bash are used, such as arrays and parameter expansion. Other shells might be compatible, but have not been tested.
- [GNU core utilities](https://www.gnu.org/software/coreutils/). This concerns tools such as cut, head, tail and tr for basic text processing, and base64 for encoding JPEG pictures for upload.
- [GNU Grep](https://www.gnu.org/software/grep/). Used for its text matching capabilities.
- [GNU sed](https://www.gnu.org/software/sed/). Used to get information from LLDAP's configuration file if no authentication options are provided.
- [jq](https://jqlang.github.io/jq/). Used to process responses from LLDAP's GraphQL API.
- [curl](https://curl.se/). Used to communicate with LLDAP over HTTP.

# How to use
Below is a copy of LLDAP-CLI's help information.
```
LLDAP-CLI(1)

NAME
  lldap-cli - 'lldap' Administration & Configuration CLI

DESCRIPTION
  This is the main administration command that can be used for all
  interactions with LLDAP. It offers the same functionality as the
  web interface as of LLDAP v0.5.0, and offers additional functionality
  for working with custom user and group attributes as introduced in
  LLDAP v0.5.1-alpha.

ENVIRONMENT
  Environment variables can be used to provide connection information.

  LLDAP_CONFIG       - Location of lldap's configuration file to read
                       connection information from. E.g.: /etc/lldap.toml
  LLDAP_HTTPURL      - URL of lldap's web interface.
                       Default value: http://localhost:17170
  LLDAP_USERNAME     - Username to authenticate to lldap.
                       Default value: admin
  LLDAP_PASSWORD     - Password to authenticate to lldap.
  LLDAP_TOKEN        - Authentication token for lldap. Replaces username and
                       password. Has a good synergy with the login and logout
                       commands.
  LLDAP_REFRESHTOKEN - Authentication refresh token for lldap. Can be used with
                       the login command to request a token for subsequent
                       requests.
  LLDAP_HTTPENDPOINT_<AUTH|GRAPH|LOGOUT|REFRESH>
                     - Override a specific default HTTP endpoint. Useful if a
                       reverse proxy is used that rewrites URLs, for example.

SYNOPSIS
  lldap-cli [ OPTIONS... ] COMMAND SUBCOMMAND [ ARGUMENTS... ]

  This script attempts to read the necessary connection configuration from
  lldap's configuration file. This might need root permissions, and might
  fail. The configured location (default or overruled by environment variable
  is:

    /etc/lldap.toml

  OPTIONS can be used to provide or override connection and login variables.

  [ OPTIONS... ] :=
    -H <HTTPURL>       - HTTP base URL of the lldap management interface
    -D <ADMINUSERNAME> - Username of the admin account to login with
    -w <ADMINPASSWORD> - Password of the admin account to login with
    -W                 - Prompt user to enter the admin account password
    -t                 - Token, which replaces username and password
    -r                 - Refresh token, for use with login and logout commands

COMMAND := { login | logout | user | group } SUBCOMMAND ARGUMENTS

(SUB)COMMANDS
  COMMAND login :=
    lldap-cli login

    Prints a token and a refresh token. The token can be used to run multiple
    commands without re-authenticating for each command. The refresh token can
    be used with the logout command. Can only be used when providing either
    a username and password combination or a refresh token as login options.

  COMMAND logout :=
    lldap-cli logout

    Invalidates a refresh token and associated token.

  COMMAND user :=
    lldap-cli user list [uid | email] 
    lldap-cli user add <UID> <EMAIL ADDRESS> [-p <PASSWORD>|-P] \
      [-d <DISPLAYNAME>] [-f <FIRSTNAME>] [-l <LASTNAME>] [-a <AVATARJPEGFILE>]
    lldap-cli user del <UID | EMAIL ADDRESS>
    lldap-cli user update <MUTATION> <UID | EMAIL ADDRESS> <ATTRIBUTE> [VALUE]

    lldap-cli user info [UID | EMAIL ADDRESS]
    lldap-cli user attribute list <UID | EMAIL ADDRESS>
    lldap-cli user attribute values <UID | EMAIL ADDRESS> <ATTRIBUTE>

    lldap-cli user group add <UID | EMAIL ADDRESS> <GROUPNAME>
    lldap-cli user group del <UID | EMAIL ADDRESS> <GROUPNAME>
    lldap-cli user group list <UID | EMAIL ADDRESS>

    Arguments for adding a user are mostly self-explanatory.
    Option -P provides a prompt for the user to enter the new password.

    <MUTATION> := {set | clear | add | del}
    For updating a user attribute, mutation type set can only be used for
    attributes that are not lists. Mutation types add and del should be used
    for lists. Mutation type clear can be used to clear any attribute,
    including an entire list in a single command.
    For <ATTRIBUTE>, use an editable attribute name.
    Use the following command to find attribute names and their properties:
      lldap-cli schema attribute user list

  COMMAND group :=
    lldap-cli group list
    lldap-cli group add <NAME>
    lldap-cli group del <NAME>
    lldap-cli group update <set | clear | add | del> <NAME> <ATTRIBUTE> [VALUE]

    lldap-cli group info <NAME>
    lldap-cli group attribute list <NAME>
    lldap-cli group attribute values <NAME> <ATTRIBUTE>

  COMMAND schema :=
    lldap-cli schema attribute <user|group> add <NAME> <TYPE> [ATTRIBUTE OPTIONS]
    lldap-cli schema attribute <user|group> del <NAME>
    lldap-cli schema attribute <user|group> list

    lldap-cli schema objectclass <user|group> add <NAME>
    lldap-cli schema objectclass <user|group> del <NAME>
    lldap-cli schema objectclass <user|group> list

    <TYPE> := {string | integer | date_time | jpeg_photo}
    [ATTRIBUTE OPTIONS] :=
      -l - Attribute is a list
      -v - Attribute is visible
      -e - Attribute is editable

    All attributes and object classes included in LLDAP's default schema are
    hardcoded and cannot be deleted. Hardcoded attributes are listed. Hardcoded
    object classes are not listed.

EXAMPLES
  eval $(lldap-cli -D admin -w abcd1234 login)
    Login with username admin and with password abcd1234, and create
    environment variables for the resulting tokens that will automatically be
    used in subsequent commands for authentication.

  LLDAP_USERNAME=admin LLDAP_PASSWORD=abcd1234 eval $(lldap-cli login)
    Same as above, but using environment variables as input instead. Note that
    it is allowed (and recommended, but not shown in these examples) to
    surround input values in single or double quotes.

  lldap-cli user add jsmith john.smith@example.com -d "John Smith" -p hunter1
    Add the user account jsmith with email address john.smith@example.com,
    display name John Smith, and password hunter1.

  lldap-cli user update set john.smith@example.com password hunter2
    Set the account password of the user with email address
    john.smith@example.com to hunter2.

  lldap-cli group add "mail users"
    Create a group named mail users.

  lldap-cli user group add jsmith "mail users"
    Add the user account jsmith to the group mail users.

  lldap-cli schema attribute user add mailAlias string -l -v
    Add a new attribute to the user schema with the name mailAlias. This
    attribute concerns a list, which means that it can occur multiple times
    in a single LDAP record. It is also visible when queried with LDAP. The
    attribute can only be modified through LLDAP's GraphQL API (e.g. using
    LLDAP-CLI) and not LDAP, since it is not marked as editable.

  lldap-cli user update add jsmith mailAlias johnnyboy@example.com
    Adds text value johnnyboy@example.com to the attribute list mailAlias.
    Note that this value is different from John's (main) mail address, which
    is still john.smith@example.com.
```
