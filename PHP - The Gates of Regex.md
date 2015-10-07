# PHP - The Gates of Regex

_A quick guide to get you started in the perplexing world of regex_

**Published:**  2013/05/25 (May 25 2013)

*Quick Note: This guide is aimed at PHP but regex is generally universal so you can always use this guide for other programming languages.*

When one thinks of regex they normally think of a dreaded string of confusing characters created by some drunk coder one night, but in truth regex is easy once you understand it and very powerful!

Regex is designed to match strings in almost any configuration however this normally makes regex very slow and undesirable! You should only use regex if you really really need to.

## Article Overview
First we will start by explaining most of the characters used in regex and example usage for each one Then will then talk about a few of the quirks in regexs and things you should look for. Finally will jump into some code and break down each example to get an idea of how regex works.

Ready?

## Regex Cheat Sheet
I guarantee you will come back to this list, in-fact I'm doing it right now while writing this article!

| Regex   | Description                                                        | Example                   |
|---------|--------------------------------------------------------------------|---------------------------|
| `\`     | Escape character with several uses                                 | `/[0-9]\.[ab]/`
| `^`     | Used to match a character (or string) at the beginning of a string | `^foo`
| `$`     | Used to match a character (or string) at the end of a string       | `bar$`
| `.`     | Used to match a single character                                   | `[a-z].[a-z]`
| &#124;  | Used normally as a logical separator; same as either or            | `(youtube.com`&#124;`youtu.be)`
| `[...]` | Used to specify valid parameters for a section of string.          | `[a-zA-Z0-9]`
| `(...)` | Start and end of a sub pattern                                     | `(saturday`&#124;`sunday)`
| `+`     | Metacharacter; Match one or more of the preceding expression       | `[a-z]+`
| `*`     | Metacharacter; Match zero or more of the preceding expression      | `(.*\.com)`
| `?`     | Metacharacter; Match zero or one of the preceding expression       | `[a-z]?`
| `{...}` | Metacharacter; Match a specific amount of occurrences              | `[a-z]{3}`
| `\s`    | Any whitespace character; Treats string as single lines            | `/[\s,]+/`
| `\S`    | Any non-whitespace character                                       | `/[\S,]+/`
| `\d`    | Any digit                                                          | `/(\d{3})-(\d{3})-(\d{4})/`
| `\D`    | Any non-digit                                                      | `/(\D)/`
| `\w`    | Any word character, letter, number, underscore; or `[A-Za-z0-9_]`  | `(\w)`
| `\W`    | Any non-word character, letter, number, underscore                 | `(\W)`
| `\b`    | Any word boundary                                                  | `/\bhellob\b/`
| `\i`    | Case insensitive                                                   | `/regexp/i`
| `\m`    | Make dot match newlines; changes the behavior of the `^` and `$`   | `/regexp/m`
| `\x`    | Ignore whitespace in regex                                         | `/regexp/x`

## A few discrepancies you need to remember

### Backslashes and quotes

If you use the backslash (\) to indicate your regex statement and you are using double quotes ("") instead of single quotes ('') then you need to use two back slashes instead of one.

```regex
$single = '\.';
$double = "\\.";
```

### Characters you need to escape

You need to escape the following characters.

```regex
. - \ + * ? ^ $ [ ] ( ) { } < > = ! | :
```

### Regex has multiple delimiters

In general regex use \, #, and ~ as delimiters for a regex string. Here is an example:

```regex
/foo bar/
#[a-zA-Z]#
+regex+
%[\w]%
```

### ?: Negate a sub-pattern

The ?: rule is used to negate a sub-pattern when reporting back in php. Say I wanted to match a url, I could do:

```regex
/(http|https|)(:\/\/|)(www.|)(.*)(.)([a-zA-Z0-9]{3,4})/i
```

However when I call back the string in php using something like preg_match_all or preg_match it would return the whole string: `http://www.example.tld`. But if I only wanted the domain name and tld I would need to call each part separately.

For example:

```php
<?php
$string = 'http://www.website.tld';
$pattern = '/(http|https|)(:\/\/|)(www.|)(.*)(.)([a-zA-Z0-9]{3,4})/i';

preg_match($pattern, $string, $matches);
echo $matches[0]; // Would return http://www.website.tld

echo $matches[4] . $matches[5] . $matches[6]; // Would return website.tld
```

Instead what I can do is negate a sub-pattern by using ?:

```regex
/(?:http|https|)(?::\/\/|)(?:www.|)(.*)(.)([a-zA-Z0-9]{3,4})/i
```

Example:

```php
<?php
$string = 'http://www.website.tld';
$pattern = '/(?:http|https|)(?::\/\/|)(?:www.|)(.*)(.)([a-zA-Z0-9]{3,4})/i';

preg_match($pattern, $string, $matches);
echo $matches[0];// Would return website.tld

// No longer needed!
// echo $matches[4] . $matches[5] . $matches[6];
```

## Diving into some code

Now lets breaking down a regex expression and understand it.

The Code:

```php
<?php
$string = 'test@website.tld';
$pattern = '/([a-zA-Z0-9]+)(@)([a-zA-Z0-9]+)(.)([a-zA-Z0-9]{3,4})/i';

preg_match($pattern, $string, $matches);
echo '<pre>';
print_r($matches);
echo '</pre>';
```

This is a regex expression to match an email address, for example "test@website.tld" with a 3 ~ 4 long domain (com, net, org, info, etc...).

Lets take a closer look at:

```regex
/([a-zA-Z0-9]+)(@)([a-zA-Z0-9]+)(.)([a-zA-Z0-9]{3,4})/i
```

A bit scary right? Lets break it down into sections:

```
/                                 Start the delimiter
(                                 Start a sub-pattern
    [a-zA-Z0-9]                   Match any characters that are alphanumeric
    +                             Continue the expression before until next sub-pattern
)                                 End a sub-pattern
(                                 Start a sub-pattern
    @                             Look for an @ sign
)                                 End a sub-pattern
(                                 Start a sub-pattern
    [a-zA-Z0-9]                   Match any characters that are alphanumeric
    +                             Continue the rule before until next sub-pattern
)                                 End a sub-pattern
(                                 Start a sub-pattern
     .                            Look for a period (.) sign
)                                 End a sub-pattern
(                                 Start a sub-pattern
    [a-zA-Z0-9]                   Match any characters that are alphanumeric
    {3,4}                         For the rule before do this for the next 3-4 characters
)                                 End a sub-pattern
/                                 End the delimiter
i                                 State that the string is NOT case-sensitive
```

As you can see I broke down the code almost characters by character but a lot of the above code was repeated.

The only thing that makes regex so complicated is that all the rules stringed together can create on ugly expression.

Now that was an easy string, lets try something a bit harder. Let's match a youtube url however we will also match every possible youtube url possible.

I wrote this function a while back that will find every possible youtube url/id on a page:

```php
<?php
/*
 * searchForYouTubeIds
 * 
 * This method is used to search YouTube ids in a string.
 * It can find normal urls, shorten urls (youtu.be), watch?=, /v/, watch&=, /embed/, 
 * watch?somestring=asdasd&v=<video id>, iframes and <embed> tags.
 * 
 * @param string $content The string to scan
 * @return array An array of urls and ids
 */
function searchForYouTubeIds($content = null) {
    preg_match_all("/(?:http|https|)(?::\/\/|)(?:www.|)(?:youtube.com|youtu.be)(?:\/embed\/|\/v\/|\/watch|\/)(\?[a-z+&\$_.-][a-z0-9;:@&%=+\/\$_.-]*|[\w-]{10,12})/i", $content, $foundIDs);

    foreach($foundIDs[1] as $ids) {
        parse_str(parse_url($ids, PHP_URL_QUERY), $new_ids);
        if(!empty($new_ids['v'])) {
            $newUrls[] = $new_ids['v'];
        } else {
            $newUrls[] = $ids;
        }
    }
    
    $data['url'] = $foundIDs[0];
    $data['id'] = $newUrls;
    
    return $data;
}
```

and you can call it by doing:

```php
<?php
echo '<pre>';
print_r(searchForYouTubeIds($content));
echo '</pre>';
```

This will return an array of data.

If you want to try the expression save this into a php file:

```php
<?php
$content = "
<h1>YouTube Search for all YouTube IDS</h1>
<hr />
<p>This is an example of most if not all new YouTube ids in diffrent formats.</p>
<p>YouTube ids are 11 characters long and contain (a-zA-Z0-9-_).</p>
<hr />
Hellow World, I just want to share this YT video http://www.youtube.com/watch?v=TtWvpDFzGE0 and this one: http://www.youtube.com/watch?v=fdi7Lqk8Plo
<br />
<br />
http://www.youtube.com/watch?v=TtWvpDFzGE0<br />
http://youtube.com/watch?v=TtWvpDFzGE0<br />
youtube.com/watch?v=fdi7Lqk8Plo<br />
<br />
http://www.youtube.com/v/fdi7Lqk8Plo<br />
http://youtube.com/v/fdi7Lqk8Plo<br />
youtube.com/v/fdi7Lqk8Plo<br />
<br />
http://www.youtube.com/embed/fdi7Lqk8Plo<br />
http://youtube.com/embed/fdi7Lqk8Plo<br />
youtube.com/embed/fdi7Lqk8Plo<br />
<br />
http://www.youtu.be/fdi7Lqk8Plo<br />
http://youtu.be/fdi7Lqk8Plo<br />
youtu.be/fdi7Lqk8Plo<br />
<br />
http://www.youtube.com/watch?foo=bar&v=aswegvfedfy<br />
http://www.youtube.com/watch?foo=bar&v=fdi7Lqk8Plo<br />
<br />
Don't forget about URLS like this: http://www.youtube.com/watch?v=fdi7Lqk8Plo&feature=autoshare&version=3&autohide=1&autoplay=1
<br />
<br />
<p>Oh and embeds as well!</p>
<iframe width=\"560\" height=\"315\" src=\"http://www.youtube.com/embed/TtWvpDFzGE0\" frameborder=\"0\" allowfullscreen></iframe>
";

echo $content;

/*
 * searchForYouTubeIds
 * 
 * This method is used to search YouTube ids in a string.
 * It can find normal urls, shorten urls (youtu.be), watch?=, /v/, watch&=, /embed/, 
 * watch?somestring=asdasd&v=<video id>, iframes and <embed> tags.
 * 
 * @param string $content The string to scan
 * @return array An array of urls and ids
 */
function searchForYouTubeIds($content = null) {
    preg_match_all("/(?:http|https|)(?::\/\/|)(?:www.|)(?:youtube.com|youtu.be)(?:\/embed\/|\/v\/|\/watch|\/)(\?[a-z+&\$_.-][a-z0-9;:@&%=+\/\$_.-]*|[\w-]{10,12})/i", $content, $foundIDs);

    foreach($foundIDs[1] as $ids) {
        parse_str(parse_url($ids, PHP_URL_QUERY), $new_ids);
        if(!empty($new_ids['v'])) {
            $newUrls[] = $new_ids['v'];
        } else {
            $newUrls[] = $ids;
        }
    }
    
    $data['url'] = $foundIDs[0];
    $data['id'] = $newUrls;
    
    return $data;
}

echo '<h1>Function retuns array of urls and ids</h1>';

echo '<pre>';
print_r(searchForYouTubeIds($content));
echo '</pre>';
```

Alright enough with the blocks of text let's break down this expression!

```regex
/(?:http|https|)(?::\/\/|)(?:www.|)(?:youtube.com|youtu.be)(?:\/embed\/|\/v\/|\/watch|\/)(\?[a-z+&\$_.-][a-z0-9;:@&%=+\/\$_.-]*|[\w-]{10,12})/i
```

Looks super scary right?

Let's just break it down again but lets do it sub-pattern by sub-pattern

```
/                                   Start the delimiter
(                                   Start a sub-pattern
    ?:                              The ?: says negate this pattern
    http|https|                     Match http, https, or nothing
)                                   End the sub-pattern
(                                   Start a sub-pattern
    ?:                              The ?: says negate this pattern
    :\/\/|                          Match :// or nothing
)                                   End the sub-pattern
(                                   Start a sub-pattern
    ?:                              The ?: says negate this pattern
    www.|                           Match www. Or nothing
)                                   End the sub-pattern
(                                   Start a sub-pattern
    ?:                              The ?: says negate this pattern
    youtube.com|youtu.be            Match two type of domains
)                                   End the sub-pattern
(                                   Start a sub-pattern
    ?:                              The ?: says negate this pattern
    \/embed\/|\/v\/|\/watch|\/      Match /embed/ or /v/ or /watch/ or /
)                                   End the sub-pattern
(                                   Start a sub-pattern
    \?                              Escape the question mark (?)
    [a-z+&\$_.-]                    Match all the defined characters
    [a-z0-9;:@&%=+\/\$_.-]          Match all the defined characters
    *                               Match all that follow previous rule
    |                               OR (logic separator)
    [\w-]                           Match alphanumeric and "-".
    {10,12}                         Must also be 10-12 characters long
)                                   End the sub-pattern
/                                   End the delimiter
i                                   Can be case-insensitive
```

Hope your head is not spinning yet, but that was one complicated expression. Hopefully you can see how each part of the expressions works.

## Closing Remarks
I really can't saying any else about regex. If you want to really become good at writing regex expressions you need to dive in and write some your self. Don't forget you have that nifty cheat sheet at the top of this article to help you out.

## Resources, Discussions & Papers
Below is a list of resources I used to create this tutorial, please visit them to learn more information

**PHP.net manual or PCRE patterns:**
* http://www.php.net/manual/en/reference.pcre.pattern.syntax.php

**Regular Expression Information**
* http://www.regular-expressions.info 
