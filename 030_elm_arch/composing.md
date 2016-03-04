# Composing

One of the big benefits from using the Elm architecture is the way it handles composition of components. To understand this let's build an example:

- We will have a parent component `App`
- And a child component `Widget`

## Children component

Let's begin with the children component. This is the code for __Widget.elm__:

```elm
module Widget (..) where

import Html exposing (Html)
import Html.Events as Events


-- MODEL


type alias Model =
  { count : Int
  }


initialModel : Model
initialModel =
  { count = 0
  }


type Action
  = Increase

-- VIEW


view : Signal.Address Action -> Model -> Html
view address model =
  Html.div
    []
    [ Html.div [] [ Html.text (toString model.count) ]
    , Html.button
        [ Events.onClick address Increase
        ]
        [ Html.text "Click" ]
    ]



-- UPDATE


update : Action -> Model -> Model
update action model =
  case action of
    Increase ->
      { model | count = model.count + 1 }
```

This component should be straighforward to understand as it is nearly identical to the application that we made in the last section. 

This component has:

- a `Model`
- actions i.e. `Increase`
- a `view` that displays the counter and a button for increasing it
- an `update` function that responds to the `Increase` action and changes the model

Note how the component only knows about things declared here. Both `view` and `update` only use types declared within the component (`Action` and `Model`).

## The parent component

This is the code for the parent component.

```elm
module Main (..) where

import Html exposing (Html)
import StartApp.Simple
import Widget as Widget


-- MODEL


type alias AppModel =
  { widgetModel : Widget.Model
  }


initialModel : AppModel
initialModel =
  { widgetModel = Widget.initialModel
  }


type Action
  = WidgetAction Widget.Action

-- VIEW


view : Signal.Address Action -> AppModel -> Html
view address model =
  Html.div
    []
    [ Widget.view (Signal.forwardTo address WidgetAction) model.widgetModel
    ]



-- UPDATE


update : Action -> AppModel -> AppModel
update action model =
  case action of
    WidgetAction subAction ->
      let
        updatedWidgetModel =
          Widget.update subAction model.widgetModel
      in
        { model | widgetModel = updatedWidgetModel }



-- START APP


main : Signal.Signal Html
main =
  StartApp.Simple.start
    { model = initialModel
    , view = view
    , update = update
    }
```

Let's break this down.

### Model

```elm
type alias AppModel =
  { widgetModel : Widget.Model
  }
```

The parent component has its own model. One of the attribute on the model contains the `Widget.Model`. Note how this parent component doesn't need to know about what `Widget.Model` is.

```elm
initialModel : AppModel
initialModel =
  { widgetModel = Widget.initialModel
  }
```

When creating the initial application model, we simply call `Widget.initialModel` from here.

If you were to have multiple children components you would do the same for each, for example:

```
initialModel : AppModel
initialModel =
  { navModel = Nav.initialModel,
  , sidebarModel = Sidebar.initialModel,
  , widgetModel = Widget.initialModel
  }
```

Or we could have multiple children components of the same type:

```
initialModel : AppModel
initialModel =
  { widgetModels = [Widget.initialModel]
  }
```

### Actions

```elm
type Action
  = WidgetAction Widget.Action
```

We use an __union type__ that wraps `Widget.Action` to indicate that an action belongs to that component. 

In an application with multiple chilren components we may have something like:

```elm
type Action
  = WidgetAction Widget.Action
```