# Syntax specifications

## 1. Overview

*Chatette* is a Python script that can generate an input file for *Rasa NLU* from a file containing templates written in a Domain Specific Language. This document describes this DSL.

The DSL is a superset of *Chatito*'s DSL, described [here](https://github.com/rodrigopivi/Chatito/blob/master/spec.md).

Here is an example of such a template file.

```
; This template file defines examples of sentences people would say to introduce themselves
%[&introduce](2)
    ~[hi] ~[i am] @[name][, nice to meet you?]
    ~[hi] my name{'s/ is} @[name]

~[hi]
    hi
    hello
    howdy

~[i am]
    {i/I} am
    {i/I}'m

@[name]
    John
    Robert = Robert
    Bob = Robert
```

The generated file will contain example data and will thus look as the one shown hereafter. This data is made up of examples sentences which are labelled as having a certain intent and optionally contain one or several entities corresponding to the value of a slot. There is also a support to list synonyms for entity value. Read about *Rasa NLU*'s data format [here](http://rasa.com/docs/nlu/0.13.1/dataformat/) for more information.
```json
{
  "rasa_nlu_data": {
    "common_examples": [
      {
        "entities": [
          {
            "end": 13,
            "entity": "name",
            "start": 7,
            "value": "Robert"
          }
        ],
        "intent": "introduce",
        "text": "Hi i'm Robert"
      },
      {
        "entities": [
          {
            "end": 19,
            "entity": "name",
            "start": 16,
            "value": "Robert"
          }
        ],
        "intent": "introduce",
        "text": "Howdy my name's Bob"
      }
    ],
    "entity_synonyms": [
      {
        "synonyms": [
          "Robert",
          "Bob"
        ],
        "value": "Robert"
      }
    ],
    "regex_features": []
  }
}
```

Template files can be encoded in any charset, while JSON files will always be UTF-8 encoded.

## 2. Language syntax

As visible in the example shown above, a file is composed of several **definitions**, made up of what we will reference as a **declaration** and a **set of rules**, one rule per line.
The declaration is the unindented part, while the set of rules are indented. The indentation must be coherent inside a rule, i.e. each rule in a definition must be indented in the same way. However, different indentations may be used for different definitions.

Everything that follows a semi-colon (`;`) is considered to be a comment and is thus ignored.

The structure of such a file can thus be represented as follows:
```
; Whole line comment
DECLARATION1
    RULE1
    RULE2
DECLARATION2  ; Other comment
  RULE1
  RULE2
```

Semi-colons are not the only characters with a special meaning in the DSL. If you want to use special characters as normal characters, they should be escaped by prepending a backslash (`\`) to them. This is true anywhere in the template documents, except in comments.

Here is an exhaustive list of characters which should be escaped:

|   |   |
|---|---|
| Semi-colon | `;` |
| Percent | `%` |
| Tilde | `~` |
| At | `@` |
| Question mark | `?` |
| Hashtag | `#` |
| Slash | `/` |
| Dollar sign | `$`
| Ampersand | `&` |
| Square brackets | `[` and `]` |
| Curly braces | `{` and `}` |
| Equal symbol | `=` |
| Pipe | `\|` |
| Backslash | `\` |

Note that not escaping these characters might not be problematic in certain cases, but it is more prudent to do it in all cases. Escaping a character outside of this list will simply be ignored by the parser which will act as if there was no backslash there.

### 2.1. Rules

As we said, a definition contains a set of rules. A rule is made of "tokens" which are either words (or group of words) or references to definitions.
A rule is used to generate example: each token of the rule, being able to generate one or several words, generates them in sequence. However, the generation of most of those tokens can be adapted using modifiers.

We list and explain each and every possible token hereafter.

#### 2.1.1. Words

Words are tokens that can only generate one word. This kind of token cannot take any modifier.

For example, the rule `this is a rule`, which contains 4 word tokens, will *always* generate the sentence `this is a rule`.

#### 2.1.2. Word groups

Word groups are tokens that are a little more adaptable. As the name suggests, they are a group of words grouped within square brackets.
Such tokens will generate the exact string that is enclosed in the square brackets, unless some modifiers adapts their generation.

A word group can take three modifiers:

- A case generation modifier

  Adding an ampersand `&` as the first character in the square brackets will make the word group choose to generate its string with either a leading uppercase or lowercase letter at random.

  For example, the word group `[&hello I'm a test]` will generate `hello I'm a test` 50% of the time and `Hello I'm a test` 50% of the time.

- A random generation modifier

  Adding a question mark `?` after the word group's string will make the word group either generate something or nothing.

  For example, `[test?]` will generate `test` 50% of the time and nothing 50% of the time.

  It is possible to give a name to such a modifier. Then, each token that will have a random modifier with the same name in the rule will generate something if and only if the other tokens with the same modifier generated something, and vice-versa.

  For example, the rule `Hey [I'm a?name] pretty [test?name]` will generate either `Hey I'm a pretty test` or `Hey pretty`.

- A percentage for the random generation modifier

  When adding a random generation modifier as shown above, the generation defaults to generating the token 50% of the time and not generating it 50% of the time. This number can be changed by giving the percentage of the time the token should generate something, without the percent symbol `%` and separated from the random generation modifier by a slash `/`.

  Thus, the rule `[test?/80]` will generate `test` 80% of the time, and will generate nothing 20% of the time.

  Of course, a name for the random generation can still be provided between the question mark and the slash.

#### 2.1.3. Aliases

#### 2.1.4. Slots

#### 2.1.5. Intents

#### 2.1.6. Choices

### 2.2. Declarations

As we said, each definition starts with a declaration which is made in the following way: a special character to distinguish the different types of definitions, followed by an identifier in-between square brackets.
Identifier can be made of any characters, including whitespaces. To use characters from the previous table in identifier, escape them. The only forbidden characters are the line feed (`\n`) and carriage return (`\r`).

The identifier of a definition will be the way to reference this definition elsewhere in the template files. Note that a declaration doesn't have to be done before being referenced and doesn't even have to be done in the same file as its references, as long as the declaration exists somewhere.

Some modifiers may be added to declarations to affect what they will generate. Those modifications are made using special characters, put besides the identifier (inside the brackets).
Three modifiers for declarations are currently supported:

- Case generation (in short *casegen*) using `&`

  If this modifier is used, everything that is generated by this definition will randomly begin with an uppercase or lowercase letter. The ampersand must be the first thing inside the square brackets, for example:
  ```
  ~[&declaration]
    ...
  ```

- Variation naming using `#` and a variation identifier

   This modifier can be used to differentiate different forms of the same declaration. You can thus make several definitions for the same identifier and use them in different places. For example, you might want to make a singular and a plural variation for a single definition, as shown next:
   ```
   ~[def#singular]
    ...
   ~[def#plural]
    ...
   ```

   The hashtag must be placed after the declaration name and the variation identifier must be placed after the `#`. An empty variation name is not allowed.

 - Argument support using `$` and an argument name

    This modifier allows to tell that certain parts of the rules can be replaced by a provided word or group of word. This is very useful when you have two definitions that would be the same if there wasn't a certain word inside that was used.

    The dollar sign must be placed after the declaration name and the identifier for the argument has to be placed after that. As for any other identifiers, arguments identifiers can be made up of any characters except line breaks (and special characters should be escaped).

    Inside rules, you would reference the argument simply by writing it, prepended by a dollar sign.
    For example:
    ```
    ~[greet$NAME]
        hello $NAME
    ```

There are three different types of definitions, identified by the leading special character of their declaration: alias definitions, slot definitions and intent definitions.

#### 2.2.1. Alias definitions

An alias is defined as a

#### 2.2.2. Slot definitions

#### 2.2.3. Intent definitions