## kotlin-styled

Kotlin wrappers for [styled-components](https://www.styled-components.com/) and 
[inline-style-prefixer](https://github.com/rofrischmann/inline-style-prefixer).

### Installation

1. `npm i @jetbrains/kotlin-css @jetbrains/kotlin-css-js @jetbrains/kotlin-styled inline-style-prefixer styled-components`

2. `npm run gen-idea-libs`

See the [Bintray page](https://bintray.com/kotlin/kotlin-js-wrappers/kotlin-styled) for Maven and Gradle installation 
instructions.

### Getting Started

`kotlin-styled` is a great fit for applications built using `kotlin-react`. It gives you not only a type-safe way of 
authoring stylesheets, but it also takes care of adding vendor prefixes for your CSS rules, assembling stylesheets, 
and injecting them into the DOM.

If you are not familiar with [styled-components](https://www.styled-components.com/) or CSS-in-JS in general, now would 
be a good time to [inform yourself about the concept](https://hackernoon.com/all-you-need-to-know-about-css-in-js-984a72d48ebc), 
because `kotlin-styled` implements this exact idea... in Kotlin.

When using just `kotlin-react` you would create a regular CSS file and then you would reference CSS classes from Kotlin 
like this:

```kotlin
fun RBuilder.div() {
    div("some-class") {
        +"Hello world!"
    }
}
```

With `kotlin-styled` you never have to leave Kotlin:

```kotlin
fun RBuilder.div() {
    styledDiv {
        css {
            padding(vertical = 16.px)
	        
            backgroundColor = Color.green
        }    
        +"Hello world!"
    }
}
```

While you can mix markup and styles in one-off scenarios like the example above, most times you would probably want to 
have them separated to enable code reuse:

```kotlin
object ComponentStyles : StyleSheet("ComponentStyles", isStatic = true) {
    val wrapper by css {
        padding(vertical = 16.px)
        
        backgroundColor = Color.green
    }
}

fun RBuilder.div() {
    styledDiv {
        css {
            +ComponentStyles.wrapper
        }
        +"Hello world!"
    }
}
```

The latter is much easier to debug in the browser as well: when inspecting the element you'll see readable class names, 
e.g. `class="ComponentStyles-wrapper"` rather than generated ones.

### CSS Properties

The DSL supports most common CSS properties and values, including animations, transforms, shadows, and flexbox. 
**SVG properties as well as CSS grids are not supported yet, contributions are welcome**. 
However, you can use `put("property", "value")` syntax for any unsupported property:

```kotlin
fun RBuilder.div() {
    styledDiv {
        css {
            put("will-change", "transform")
        }
    }
}
```

### CSS Selectors

The DSL allows you to use most CSS selectors. See 
[CSSBuilder](https://github.com/JetBrains/kotlin-wrappers/blob/master/kotlin-css/src/main/kotlin/kotlinx/css/CSSBuilder.kt#L35)
for more details. **Contributions are welcome**.

```kotlin
object ComponentStyles : StyleSheet("ComponentStyles", isStatic = true) {
    // Example of an ".element:hover" selector
    val element by css {        
        backgroundColor = Color.green
        
        hover {
            backgroundColor = Color.red
        }
    }
    
    // Example of a ".wrapper > *" selector
    val wrapper by css {
        children {
            // CSS properties
        }
    }
    
    // Example of a ".wrapper > div" selector
    val wrapper by css {
        children("div") {
            // CSS properties
        }
    }
    
    // Example of a ".wrapper:hover .inner" selector 
    val wrapper by css {
        // CSS properties
    }
    
    val inner by css {
        backgroundColor = Color.green
        
        ancestorHover("${ComponentStyles.name}-${ComponentStyles::wrapper.name}") {
            backgroundColor = Color.red
        }
    }        
}

fun RBuilder.div() {
    styledDiv {
        css {
            +ComponentStyles.element
        }
        +"An element"
    }

    styledDiv {
        css {
            +ComponentStyles.wrapper
        }
        styledDiv {
            css {
                +ComponentStyles.inner
            }
            +"Inner element"
        }
    }
}
```

### Global Styles

To create a global stylesheet use the `CSSBuilder` class and the `StyledComponents.injectGlobal()` function:

```kotlin
val styles = CSSBuilder().apply {
    body {
        margin = 0.px
        padding = 0.px
    }
}

StyledComponents.injectGlobal(styles.toString())
```