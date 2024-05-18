# Theme Sections


## Summary


This rfc proposes to add a field `section` to the `Theme` enum.
```rust
enum Section {
   TopBar,
   Card,
   Body,
   Drawer,
   ...
}
```


## Motivation

Imagine you want to add an icon in an elevated Card, and a drawer. It is likely that the background color of these "sections" will be different. Yet, currently, the colors of the button will be the same. This change will allow widgets to change their colors based on the section they are in. This would fix all contrast issues out of the box.


## Guide-level explanation

### For widgets

The style function signature will not change. You will be able to know the section you're in and change properties accordingly. Example:


```rust
fn style(theme: Theme, status: Status) -> Style {
   let mut style = Style::default();
   if theme.section() == Section::Card {
       style.text = theme.palette().on_background_card;
   }
   style
}
```


We could also define well known widgets to structure an Iced app.
For example, a [drawer](https://m3.material.io/components/navigation-drawer/overview) which takes two `Element`s. In its draw function, it could change the theme section like this:


```rust
fn draw(
   &self,
   tree: &Tree,
   renderer: &mut Renderer,
   theme: &Theme,
   ...
) {
  
   let drawer_theme = theme.with_section(Section::Drawer);
   let body_theme = theme.with_section(Section::Body);


   self.content
           .as_widget()
           .draw(tree, renderer, &drawer_theme, &style, layout, cursor, viewport);
   self.content
       .as_widget()
       .draw(tree, renderer, &body_theme, &style, layout, cursor, viewport);
  
}
```


## Implementation strategy


The implementation for this is quite simple with the new theme API.
- change the `Theme` enum to a struct.
- add a `section` field to it.
- add a `kind` field for the previous `Theme` enum.


functions added to `Theme`:
- `pub fn section(&self) -> &Section`
- `pub fn kind(&self) -> &ThemeKind`
- `pub fn with_section(&self, section: Section) -> Theme`




But the interesting and difficult task is to have a coherent structure and style across Iced app and widgets. Ei, what section and widgets we expose to structure an app, and what colors to use.
This would probably require changing the color palette to match the set of sections exposed.


## Drawbacks


- Some could argue that it is the responsibility of the developer to make a nice UI. However, I disagree. I believe a quality UI toolkit should inherently prevent issues with contrast or coherence in your app by default. Similarly to the Rust api that tends to be impossible to miss use.
- It could lead to apps that look like each other. But I think it is partially true, since this would only expose building blocks for your UI. Also, I kinda like that toolkits have their own identity. Like you directly know if an app was made with qt or gtk.
- Users will not be able to define their own section.
- This kinda increases the number of possible combinations of styles for a widget, which is already quite big. But you don’t have to define a custom style for every section. And we could imagine helper functions for the palette.




## Rationale and alternatives


- Why is this design the best in the space of possible designs?
- What other designs have been considered and what is the rationale for not choosing them?
- What is the impact of not doing this?




## [Optional] Prior art


Some jetpack compose components:
- [scaffold](https://www.composables.com/material3/scaffold)
- [cards](https://m3.material.io/components/cards/overview)
- [dialogs](https://m3.material.io/components/dialogs/overview)
- [navigation-bar](https://m3.material.io/components/navigation-bar/overview)
- [navigation-drawer](https://m3.material.io/components/navigation-drawer/overview).


The material3 color palette:
- [how to use it](https://m3.material.io/styles/color/roles)
- [all the fields](https://developer.android.com/reference/kotlin/androidx/compose/material3/ColorScheme#public-properties)


## Unresolved questions


- What sections should we expose ?


## [Optional] Future possibilities


More and more sections.
