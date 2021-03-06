# RequireWhat

A Chrome extension that tells you what a website's password requirements are so you don't have to guess

## Planning

This project is still in early development, so I'm just sort of figuring out how it's going to go...

### Purpose

There are a few goals I have for this project. I want a user of my extension to be able to go to any website and register without having to struggle to craft an acceptable password. If the rules are simple, they should just be able to read them and understand what is required. If the rules are too complex to spell out in plain English (for example, if something like zxcvbn is used to score the password), then I should at least be able to give clear, concise feedback about exactly what the issue is when a password is rejected.

I also want the user to be able to return to a website and quickly remember what password they used without having to play guess-and-check. Yes, of course, remembering and reusing passwords is horrible security practice, but not every user cares about their password on every little site they visit.

I'd also like for password generators (such as the ones built into many password managers) to be able to easily craft a secure password that will work on any given website. Perhaps eventually the extension might include its own password generator?

### Timeline
This is just a hobby project, so I don't have any deadlines or detailed planning, but I have a rough idea of what needs to be done in what order:

* **Figure out the specifications.** I need to examine as many websites as possible to get a sense of all the types of password rulesets that exist in the world. This will allow me to flesh out the config format as much as possible so that hopefully it doesn't need major restructuring later on. (I want the rules to be defined in a machine-readable format and avoid written-out English in the configs as much as possible -- it gives me more flexibility down the line.)
* **Build an MVP of the extension.** It doesn't have to be pretty, it doesn't have to be optimized; it just needs to work. Basically it should be a little button (a "page action" in Chrome jargon) that shows up on any site in our database, and prints out the rules for that site in a popover in basic English.
* **Promote it.** Once I have something that is marginally presentable, I need to get it out on Reddit and Hacker News and see if people like it and will contribute to it. This project can't really succeed if there isn't a decent number of people contributing new site configs, so this is the make-or-break point.
* **Clean it up.** Polish the MVP into something that can grow a little bigger. Optimize the code a bit (once there are a LOT of site configs, efficient code will be crucial). Make it look pretty. Add the ability for it to examine/grade your password as you type it on the webpage.
* **Sky's the limit?** If we get this far, I think there are some neat features that could improve it even more. Add a password generator. Get people to localize it into other languages. Add recommendations for making your passwords more secure (rather than merely making them comply with the rules).

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
    "password_rules": [
        {
            "@type": "length",
            "@rule": "require",
            "min": 10,
            "max": 20
        },
        {
            "@type": "zxcvbn",
            "@rule": "require",
            "score_min": 3
        },
        {
            "@type": "personal",
            "@rule": "deny",
            "items": [
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
