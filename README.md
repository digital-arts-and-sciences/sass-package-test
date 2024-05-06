# DAS Theming system

This package uses the new `pkg::` syntax of SASS.

## Why
You're writing CSS wrong.

Generally, its a pretty safe bet.  

React systems tend to integrate style directly in-place with code, which is a general fail.

When building brand systems, it isn't important that the 'body font is 16, and header font is 20, what is important is that the header font is 1.25x the size of the body font... it is all about ratio, not direct numbers.

For this reason, we eschew things like REM.

"But REM is more predicatable" is a very valid argument -- if you're writing your CSS wrong.  Properly written, where size structures cascade properly along elements, is the way to go.

Some things to note:

* Rarely use rems.  They should only be used in very specific use cases
* Same thing for PX, and there's even less reason to use them except for cases like border, where you're going to want something like 'exactly one pixel'.  Instead, specify the dimension in rem.  For instance:  A text-shadow should be releative to the size of the text, not some fixed size... so specifying in EM allows you to scale up the font and the shadow simultaneously.  Same for things like border-radius -- if you make a box bigger, its border-radius should be bigger as well.
* CSS Variables are great, and should absolutely be used.  But using them in a general fashion is not only tricky, it winds up just making all of your styles root-level items.  Its great for re-usable systems, not great for other use cases.
* Float.  You can leave now.  This project is for CSS professinals in the modern era.
* Flex.  Except for the one or two things it does well, just use grid, which is 100x more extensible.

## Structure
This is a BEM-style structure, not just inline style shorthand like Tailwinds or Bootstrap.

## The sandwich model

A sandwich model is a pretty straightforward concept... you start with a piece of bread (the base), then layer things on top of it... meat, lettuce, cheese, etc.

In our case, that translates to (bottom to top):

* Core configuration
* Brand-level configuration
* Project-level configuration

This allows users to create a brand-level configuration and use it for all projects in a brand.  While this may not be useful for groups maintaining a single site, agencies and groups that produce a volume of sites benefit greatly from increased reusability.

## Configuring

### Standard
Standard configuration is pretty straightforward.  Just pick your brand, and import the initialize for it:

```
@use "brands/tdecu/initialize";
```

From here, the system will automatically output any top-level items, such as raw selectors in your typography map.  Generally, this will mean `body` and `html` tags.


### Creating a system with project-level overrides
You can create an override for particular components 

```
@use "project/breakpoints" as projectBreakpoints;

@use "brands/tdecu/initialize" with (
    $projectBreakpoints: (
        projectBreakpoints.$config,
    )
);
```


## Make sure it works
You can run SASS from the command line.  The following test line will load the appropriate SASS paths and output a file into `test/output`.  

```
sass test/index.scss:test/output/out.css --load-path=test --load-path=src/scss
```

This test file also displays a trace of the options map (see next section).

## Previewing the merged data
The test file is currently written to debug the entire option map, which could prove useful in debugging the system.

To do this, in SASS, simple add a line like this:
```
@use "util/manager";
...
@debug manager.json();
```

`manager` is a package, and contains the `json` function. The JSON function will serialize a MAP to json and display it to screen through SASS's @debug method.

You should get trace like this:

```
src/scss/util/_manager.scss:19 Debug: EXPORTING OPTIONS AS JSON:

{
    "breakpoints": {
         "smaller": "460px",
         "small": "670px",
         "medium": "850px",
         "large": "1024px",
         "larger": "1400px",
         "largest": "1840px"
     },
     "colors": {
        "named": {
             "near-white": "#fff",
             "near-black": "#000",
             "tdecu-orange": "#d24114",
             "tdecu-orange-dark": "#b13711"
         }
     },
     "themes": {
        "selectors": {
            ".button": {
                "base": {
                     "color": "#fff",
                     "background-color": "#d24114"
                 },
                 "&:hover": {
                     "color": "#fff",
                     "background-color": "#b13711"
                 }
             }
         }
     }
}

test/index.scss:16 Debug: manager.json() complete.
```

In the example above, some of the data is actually interpolated at compile time (e.g., `themes>selectors>.button>base>color` is actually a reference to `colors>named>near-white`), so this is a very useful debugging tool to ensure that the chain working as expected.

This is mainly meant to be a debugging tool, but if a user wanted to, they could pipe into stdout and tokenize this, then actually export as JSON.  Not sure what that use case is, but it could be done.

Additional note:  Some javascript crawlers (Kewpie, et al), will have issues treating this as keypath.  That's due to the CSS selectors utilizing several characters which would otherwise be reserved.  No fix planned for this -- just a note.


## TODO
### Separation of brand
In our initial tests, we have coded brands directly into this package.  Instead, future brands should be their own package which is then merged at project level.  
