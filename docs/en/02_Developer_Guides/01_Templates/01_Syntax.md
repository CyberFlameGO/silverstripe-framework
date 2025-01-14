---
title: Template Syntax
summary: A look at the operations, variables and language controls you can use within templates.
icon: code
---

# Template Syntax

A template can contain any markup language (e.g HTML, CSV, JSON..) and before being rendered to the user, they're
processed through [SSViewer](api:SilverStripe\View\SSViewer). This process replaces placeholders such as `$Var` with real content from your
[model](../model) and allows you to define logic controls like `<% if $Var %>`.

An example of a Silverstripe CMS template is below:

**app/templates/Page.ss**

```ss
<html>
    <head>
        <% base_tag %>
        <title>$Title</title>
        <% require themedCSS("screen") %>
    </head>
    <body>
        <header>
            <h1>Bob's Chicken Shack</h1>
        </header>

        <% with $CurrentMember %>
            <p>Welcome $FirstName $Surname.</p>
        <% end_with %>

        <% if $Dishes %>
        <ul>
            <% loop $Dishes %>
                <li>$Title ($Price.Nice)</li>
            <% end_loop %>
        </ul>
        <% end_if %>

        <% include Footer %>
    </body>
</html>
```

[note]
Templates can be used for more than HTML output. You can use them to output your data as JSON, XML, CSV or any other 
text-based format.
[/note]

## Template file location

Silverstripe CMS templates are plain text files that have an `.ss` extension and are located within the `templates` directory of
a module, theme, or your `app/` folder.

By default, templates will have the same name as the class they are used to render. So, your `Page` class will
be rendered with the `templates/Page.ss` template.

When the class has a namespace, the namespace will be interpreted as a subfolder within the `templates` path. 
For example, the class `SilverStripe\Control\Controller` will be rendered with the
`templates/SilverStripe/Control/Controller.ss` template.

If you are using template "types" like `Layout` or `Includes`, these are just folders which you need
to append to your template file location. One exception is the `<% include %>` template tag,
where you need to leave out the `Includes/` folder.

## Variables

Variables are placeholders that will be replaced with data from the [DataModel](../model/) or the current 
[Controller](../controllers). Variables are prefixed with a `$` character. Variable names must start with an 
alphabetic character or underscore, with subsequent characters being alphanumeric or underscore:

```ss
$Title
```

This inserts the value of the Title database field of the page being displayed in place of `$Title`. 

Variables can be chained together, and include arguments.

```ss
$Foo
$Foo(param)
$Foo.Bar
```

These variables will call a method / field on the object and insert the returned value as a string into the template.

*  `$Foo` will call `$obj->Foo()` (or the field `$obj->Foo`)
*  `$Foo(param)` will call `$obj->Foo("param")`
*  `$Foo.Bar` will call `$obj->Foo()->Bar()` 

If a variable returns a string, that string will be inserted into the template. If the variable returns an object, then
the system will attempt to render the object through its `forTemplate()` method. If the `forTemplate()` method has not 
been defined, the system will return an error.

[notice]
If you wish to pass parameters to getter functions, you must use the full method name, e.g. $getThing('param'). Also, parameters must be literals, and cannot be other template variables (`$getThing($variable)` will not work)
[/notice]

[note]
For more detail around how variables are inserted and formatted into a template see 
[Formating, Modifying and Casting Variables](casting)
[/note]

Variables can come from your database fields, or custom methods you define on your objects.

**app/code/Page.php**

```php
public function UsersIpAddress()
{
    return $this->getRequest()->getIP();
}
```

**app/code/Page.ss**

```html
<p>You are coming from $UsersIpAddress.</p>
```

[note]
	Method names that begin with `get` will automatically be resolved when their prefix is excluded. For example, the above method call `$UsersIpAddress` would also invoke a method named `getUsersIpAddress()`.
[/note]

The variables that can be used in a template vary based on the object currently in [scope](#scope). Scope defines what
object the methods get called on. For the standard `Page.ss` template the scope is the current [PageController](api:SilverStripe\CMS\Controllers\ContentController\PageController) 
class. This object gives you access to all the database fields on [PageController](api:SilverStripe\CMS\Model\SiteTree\PageController), its corresponding [Page](api:SilverStripe\CMS\Model\SiteTree\Page)
record and any subclasses of those two.

**app/code/Layout/Page.ss**

```ss
$Title
// returns the page `Title` property

$Content
// returns the page `Content` property
```

## Conditional Logic

The simplest conditional block is to check for the presence of a value (does not equal 0, null, false).

```ss
<% if $CurrentMember %>
    <p>You are logged in as $CurrentMember.FirstName $CurrentMember.Surname.</p>
<% end_if %>
```

A conditional can also check for a value other than falsy.

```ss
<% if $MyDinner == "kipper" %>
    Yummy, kipper for tea.
<% end_if %>
```

[notice]
When inside template tags variables should have a '$' prefix, and literals should have quotes. 
[/notice]

Conditionals can also provide the `else` case.

```ss
<% if $MyDinner == "kipper" %>
    Yummy, kipper for tea
<% else %>
    I wish I could have kipper :-(
<% end_if %>
```

`else_if` commands can be used to handle multiple `if` statements.

```ss
<% if $MyDinner == "quiche" %>
    I don't like quiche
<% else_if $MyDinner == $YourDinner %>
    We both have good taste
<% else %>
    Can I have some of your chips?
<% end_if %>
```

### Negation

You can check if a variable is false with `<% if not %>`.

```ss
<% if not $DinnerInOven %>
    I'm going out for dinner tonight.
<% end_if %>
```

Note that you cannot combine this with other operators such as `==`.


For more nuanced check you can use the `!` operator.

```ss
<% if $MyDinner != "quiche" %>
    Lets go out
<% end_if %>
```

### Boolean Logic

Multiple checks can be done using `||`, `or`, `&&` or `and`. 

If *either* of the conditions is true.

```ss
<% if $MyDinner == "kipper" || $MyDinner == "salmon" %>
    yummy, fish for tea
<% end_if %>
```

If *both* of the conditions are true.

```ss
<% if $MyDinner == "quiche" && $YourDinner == "kipper" %>
    Lets swap dinners
<% end_if %>
```

### Inequalities

You can use inequalities like `<`, `<=`, `>`, `>=` to compare numbers.

```ss
<% if $Number >= "5" && $Number <= "10" %>
    Number between 5 and 10
<% end_if %>
```

## Includes

Within Silverstripe CMS templates we have the ability to include other templates using the `<% include %>` tag. The includes
will be searched for using the same filename look-up rules as a regular template. However in the case of the include tag
an additional `Includes` directory will be inserted into the resolved path just prior to the filename.

```ss
<% include SideBar %> <!-- chooses templates/Includes/Sidebar.ss -->
<% include MyNamespace/SideBar %> <!-- chooses templates/MyNamespace/Includes/Sidebar.ss -->
```

When using subfolders in your template structure
(e.g. to fit with namespaces in your PHP class structure), the `Includes/` folder needs to be innermost.

```ss
<% include MyNamespace/SideBar %> <!-- chooses templates/MyNamespace/Includes/Sidebar.ss -->
```

The `include` tag can be particularly helpful for nested functionality and breaking large templates up. In this example, 
the include only happens if the user is logged in.

```ss
<% if $CurrentMember %>
    <% include MembersOnlyInclude %>
<% end_if %>
```

Includes can't directly access the parent scope when the include is included. However you can pass arguments to the 
include.

```ss
<% with $CurrentMember %>
    <% include MemberDetails Top=$Top, Name=$Name %>
<% end_with %>
```

## Looping Over Lists

The `<% loop %>` tag is used to iterate or loop over a collection of items such as [DataList](api:SilverStripe\ORM\DataList) or an [ArrayList](api:SilverStripe\ORM\ArrayList) 
collection.

```ss
<h1>Children of $Title</h1>
<ul>
    <% loop $Children %>
        <li>$Title</li>
    <% end_loop %>
</ul>
```

This snippet loops over the children of a page, and generates an unordered list showing the `Title` property from each 
page. 

[notice]
The `$Title` inside the loop refers to the Title property on each object that is looped over, not the current page like
the reference of `$Title` outside the loop. 

This demonstrates the concept of [Scope](#scope). When inside a <% loop %> the scope of the template has changed to the 
object that is being looped over.
[/notice]

### Altering the list

`<% loop %>` statements iterate over a [DataList](api:SilverStripe\ORM\DataList) instance. As the template has access to the list object, 
templates can call [DataList](api:SilverStripe\ORM\DataList) methods. 

Sorting the list by a given field.

```ss
<ul>
    <% loop $Children.Sort(Title, ASC) %>
        <li>$Title</li>
    <% end_loop %>
</ul>
```

Limiting the number of items displayed.

```ss
<ul>
    <% loop $Children.Limit(10) %>
        <li>$Title</li>
    <% end_loop %>
</ul>
```

Reversing the loop.

```ss
<ul>
    <% loop $Children.Reverse %>
        <li>$Title</li>
    <% end_loop %>
</ul>
```

Filtering the loop.

```ss
<ul>
    <% loop $Children.Filter('School', 'College') %>
        <li>$Title</li>
    <% end_loop %>
</ul>
```

Methods can also be chained.

```ss
<ul>
    <% loop $Children.Filter('School', 'College').Sort(Score, DESC) %>
        <li>$Title</li>
    <% end_loop %>
</ul>
```

### Position Indicators

Inside the loop scope, there are many variables at your disposal to determine the current position in the list and 
iteration.

 * `$Even`, `$Odd`: Returns boolean, handy for zebra striping.
 * `$EvenOdd`: Returns a string, either 'even' or 'odd'. Useful for CSS classes.
 * `$First`, `$Last`, `$Middle`: Booleans about the position in the list.
    * Note: as of CMS 4.7.0 `$IsFirst` and `$IsLast` will be preferred. The original
    syntax will continue to work, but will be deprecated in a future release.
 * `$FirstLast`: Returns a string, "first", "last", "first last" (if both), or "". Useful for CSS classes.
 * `$Pos`: The current position in the list (integer).
   Will start at 1, but can take a starting index as a parameter.
 * `$FromEnd`: The position of the item from the end (integer).
   Last item defaults to 1, but can be passed as a parameter.
 * `$TotalItems`: Number of items in the list (integer).

```ss
<ul>
    <% loop $Children.Reverse %>
        <% if First %>
            <li>My Favourite</li>
        <% end_if %>

        <li class="$EvenOdd">Child $Pos of $TotalItems - $Title</li>
    <% end_loop %>
</ul>
```

[info]
A common task is to paginate your lists. See the [Pagination](how_tos/pagination) how to for a tutorial on adding 
pagination.
[/info]

### Modulus and MultipleOf

`$Modulus` and `$MultipleOf` can help to build column and grid layouts.

```ss
// returns an int
$Modulus(value, offset)

// returns a boolean.
$MultipleOf(factor, offset) 

<% loop $Children %>
<div class="column-{$Modulus(4)}">
    ...
</div>
<% end_loop %>

// returns <div class="column-3">, <div class="column-2">,
```

[hint]
`$Modulus` is useful for floated grid CSS layouts. If you want 3 rows across, put $Modulus(3) as a class and add a 
`clear: both` to `.column-1`.
[/hint]

`$MultipleOf(value, offset)` can also be utilized to build column and grid layouts. In this case we want to add a `<br>` 
after every 3rd item.

```ss
<% loop $Children %>
    <% if $MultipleOf(3) %>
        <br>
    <% end_if %>
<% end_loop %>
```

### Escaping

Sometimes you will have template tags which need to roll into one another. Use `{}` to contain variables.

```ss
$Foopx // will returns "" (as it looks for a `Foopx` value)
{$Foo}px  // returns "3px" (CORRECT)
```

Or when having a `$` sign in front of the variable such as displaying money.

```ss
$$Foo // returns ""
${$Foo} // returns "$3"
```

You can also use a backslash to escape the name of the variable, such as:

```ss
$Foo // returns "3"
\$Foo // returns "$Foo"
```

[hint]
For more information on formatting and casting variables see [Formating, Modifying and Casting Variables](casting)
[/hint]

## Scope

In the `<% loop %>` section, we saw an example of two **scopes**. Outside the `<% loop %>...<% end_loop %>`, we were in 
the scope of the top level `Page`. But inside the loop, we were in the scope of an item in the list (i.e the `Child`).

The scope determines where the value comes from when you refer to a variable. Typically the outer scope of a `Page.ss` 
layout template is the [PageController](api:SilverStripe\CMS\Controllers\ContentController\PageController) that is currently being rendered. 

When the scope is a `PageController` it will automatically also look up any methods in the corresponding `Page` data
record. In the case of `$Title` the flow looks like

	$Title --> [Looks up: Current PageController and parent classes] --> [Looks up: Current Page and parent classes]

The list of variables you could use in your template is the total of all the methods in the current scope object, parent
classes of the current scope object, and any [Extension](api:SilverStripe\Core\Extension) instances you have.

### Navigating Scope

#### Up

When in a particular scope, `$Up` takes the scope back to the previous level.

```ss
<h1>Children of '$Title'</h1>

<% loop $Children %>
    <p>Page '$Title' is a child of '$Up.Title'</p>

    <% loop $Children %>
        <p>Page '$Title' is a grandchild of '$Up.Up.Title'</p>
    <% end_loop %>
<% end_loop %>
```

Given the following structure, it will output the text.
```
	My Page
	|
	+-+ Child 1
 	| 	|
 	| 	+- Grandchild 1
 	|
 	+-+ Child 2

	Children of 'My Page'

	Page 'Child 1' is a child of 'My Page'
	Page 'Grandchild 1' is a grandchild of 'My Page'
	Page 'Child 2' is a child of 'MyPage'
```
[notice]
Additional selectors implicitely change the scope so you need to put additional `$Up` to get what you expect.
[/notice]

```ss
<h1>Children of '$Title'</h1>
<% loop $Children.Sort('Title').First %>
    <%-- We have two additional selectors in the loop expression so... --%> 
    <p>Page '$Title' is a child of '$Up.Up.Up.Title'</p>
<% end_loop %>
```

#### Top

While `$Up` provides us a way to go up one level of scope, `$Top` is a shortcut to jump to the top most scope of the 
page. The  previous example could be rewritten to use the following syntax.

```ss
<h1>Children of '$Title'</h1>

<% loop $Children %>
    <p>Page '$Title' is a child of '$Top.Title'</p>

    <% loop $Children %>
        <p>Page '$Title' is a grandchild of '$Top.Title'</p>
    <% end_loop %>
<% end_loop %>
```

### With

The `<% with %>` tag lets you change into a new scope. Consider the following example:

```ss
<% with $CurrentMember %>
    Hello, $FirstName, welcome back. Your current balance is $Balance.
<% end_with %>
```

This is functionalty the same as the following:

```ss
Hello, $CurrentMember.FirstName, welcome back. Your current balance is $CurrentMember.Balance
```

Notice that the first example is much tidier, as it removes the repeated use of the `$CurrentMember` accessor.

Outside the `<% with %>.`, we are in the page scope. Inside it, we are in the scope of `$CurrentMember` object. We can 
refer directly to properties and methods of the [Member](api:SilverStripe\Security\Member) object. `$FirstName` inside the scope is equivalent to 
`$CurrentMember.FirstName`.

### Me

`$Me` outputs the current object in scope. This will call the `forTemplate` of the object.

```ss
$Me
```

## Comments

Using standard HTML comments is supported. These comments will be included in the published site.

```ss
$EditForm <!-- Some public comment about the form -->
```

However you can also use special Silverstripe CMS comments which will be stripped out of the published site. This is useful
for adding notes for other developers but for things you don't want published in the public html.

```ss
$EditForm <%-- Some hidden comment about the form --%>
```

## Related Lessons
* [Creating your first theme](https://www.silverstripe.org/learn/lessons/v4/creating-your-first-theme-1)

## Related Documentation

[CHILDREN Exclude="How_Tos"]

## How to's

[CHILDREN Folder="How_Tos"]

## API Documentation

* [SSViewer](api:SilverStripe\View\SSViewer)
* [ThemeManifest](api:SilverStripe\View\ThemeManifest)


