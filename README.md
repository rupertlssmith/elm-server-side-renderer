# elm-server-side-renderer

This is based on code originally written by 'Noah', which can be found here:

https://github.com/eeue56/elm-server-side-renderer
                                
### Static Programs
        
This project provides some 'static' program types for Elm, for running server side rendering. A server side rendering is a function from a model to a view. Typically, the view is Html Never that is then rendered as a String. The type Html Never is used, as this is a static renderer and the Html it produces should not create events for Elm to propagate to an 'update' function.

The static program types are:
        
    type alias HtmlProgram =
        Program Value (Html Never) Never

    type alias JsonProgram =
        Program Value Value Never

    type alias StringProgram =
        Program Value String Never

The Program type used is the same as Platform.Program which is:

    Program flags model msg

In the static program variants 'flags' is used as an input type, 'model' as an output type, and msg is Never, since these programs are static and do not react to any events.

The input type is always Json.Decode.Value, meaning that a data model can be passed to a rendering as Json.

There is a program constructor 'htmlToStringProgram' that uses an 'init' function like the program constructors in Elm Platform and Html. This requires a rendering function to build Html Never from the input Value, and will render this Html as a String producing a StringProgram:

        
    htmlToStringProgram : { init : Value -> Html Never } -> StringProgram

### Interfacing with Javascript

The static program types use a custom implementation to construct their programs, which works very similarly to Platform.programWithFlags. In particular its implementation sets up a field called 'program' on the Elm module in javascript with this code:

    object['program'] = function program(flags) {
         ...
        return impl.init(result._0);
    }

The 'object' here is the Elm module that is creating the program. If a program is built in a Main.elm like this:

    port Main exposing (..)

    import Html exposing (Html)
    import Json.Encode as Encode exposing (Value)
    import ServerSide.Static exposing (StringProgram, htmlToStringProgram)

    main : StringProgram
    main =
        htmlToStringProgram { init = init }

    init : Value -> Html Never
    init model =
        view

    view : Html Never
    view =
        Html.div [] [ Html.text "hello world" ]

Then in javascript code, a reference to the program can be obtained like this:

    Elm.Main.program               

You can see in the custom javascript program, that the result of applying the 'init' function is returned. This makes 'program' a function from json to String, Html or Value. In this case the rendered html can be obtained by doing:

    var html = Elm.Main.program({});