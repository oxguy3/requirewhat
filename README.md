# RequireWhat

A Chrome extension that tells you what a website's password requirements are so you don't have to guess

## Site configuration

This extension uses a large collection of handwritten config files to define the password rules for each website. These can be found in the src/sites/ directory of this repository.

### Example

Here is a barebones basic configuration for an example website:

```json
{
    "name": "Example Site",
    "domains": [
        "example.com",
        "*.example.com"
    ],
    "links": {
        "login": "https://example.com/login",
        "logout": "https://example.com/logout",
        "register": "https://example.com/register",
        "change_password": "https://example.com/change_password",
        "reset_password": "https://example.com/reset_password"
    },
    "password_rules": [
        {
            "@type": "length",
            "@rule": "allow",
            "min": 10,
            "max": 20
        },
        {
            "@type": "zxcvbn",
            "@rule": "allow",
            "score_min": 3
        },
        {
            "@type": "personal",
            "@rule": "deny",
            "types": [
                "email",
                "username"
            ]
        }
    ]
}
```

In this example config, the site example.com requires:

* Passwords be between 10 and 20 characters (inclusive)
* Passwords must get a [zxcvbn](https://github.com/dropbox/zxcvbn) score of 3 or greater.
* Passwords must not be the user's email or username.

### Fields

At the root level of the JSON object, you must have these keys:

* `name`: A user-displayable name for the config.
* `domains`: An array of domain names this config applies to. Wildcards can be indicated with asterisks (\*)
* `password_rules`: An ordered array of rules that apply to this site's passwords. Each rule should have the following keys:
  * `@type`: The rule type as a string. See the below section "Password rule types" for possible values.
  * `@rule`: "require" or "deny"
  * any additional options relevant to the rule type

### Password rule types

Here are all the currently existing types of password rules.

#### char
Specifies requirements for the types of characters allowed in the password.

Options:

| Name | Required? | Description |
| --- | --- | --- |
| `classes` | N | An array of classes of characters that activate this rule |
| `chars` | N | A string of individual characters that activate this rule |
| `match_all` | N | If true, all classes and chars must match. (default: false) |
| `position` | N | Where must these characters be located? Options: "start", "end", "inside" (i.e. not start or end), "whole", "anywhere". (default: ["anywhere"]) |

The available classes include:

* "upper": Upper case letters
* "lower": Lower case letters
* "alpha": All letters
* "alnum": Digits and letters
* "digit": Digits
* "xdigit": Hexade­cimal digits
* "punct": Punctu­ation
* "blank": Space and tab
* "space": Blank characters
* "cntrl": Control characters
* "graph": Printed characters
* "print": Printed characters and spaces
* "word": Digits, letters and underscore

#### length
Specifies requirements for the password's length.

Options:

| Name | Required? | Description |
| --- | --- | --- |
| `min` | N | Minimum length as an integer |
| `max` | N | Maximum length as an integer |
| `count_chars` | N | If true, will count characters instead of bytes (default: false) |

#### personal
Specifies that the password has rules related to including one's own personal information in the password.

Options:

| Name | Required? | Description |
| --- | --- | --- |
| `items` | Y | An array of the types of personal info that are banned |

The `items` array can contain any of the following string values:

* "dob": Birthday
* "email": Email address
* "name": Name (first, last, full, whatever)
* "phone": Phone number
* "previous_password": A previously used password
* "username": Username

#### strings
Specifies requirements for types of strings allowed in the password.

Options:

| Name | Required? | Description |
| --- | --- | --- |
| `strings` | N | An array of individual strings that activate this rule |
| `match_all` | N | If true, all strings must match. (default: false) |
| `position` | N | Where must these strings be located? Options: "start", "end", "inside" (i.e. not start or end), "whole", "anywhere". (default: ["anywhere"]) |

The available classes include:

* "upper": Upper case letters
* "lower": Lower case letters
* "alpha": All letters
* "alnum": Digits and letters
* "digit": Digits
* "xdigit": Hexade­cimal digits
* "punct": Punctu­ation
* "blank": Space and tab
* "space": Blank characters
* "cntrl": Control characters
* "graph": Printed characters
* "print": Printed characters and spaces
* "word": Digits, letters and underscore

#### zxcvbn
Specifies that the password must achieve certain results from [zxcvbn](https://github.com/dropbox/zxcvbn).

Options:

| Name | Required? | Description |
| --- | --- | --- |
| `score_min` | N | Minimum score (0-4) |
| `score_max` | N | Maximum score (0-4) |
| `guesses_min` | N | Minimum guesses to crack |
| `guesses_max` | N | Maximum guesses to crack |
| `crack_time_min` | N | Minimum number of seconds to crack |
| `crack_time_max` | N | Maximum number of seconds to crack |
