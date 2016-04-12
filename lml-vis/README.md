# [VisUI](https://github.com/kotcrab/VisEditor/wiki/VisUI) parser for [LML](http://github.com/czyzby/gdx-lml) templates

See [gdx-lml](https://github.com/czyzby/gdx-lml/tree/master/lml). This extension allows *LML* to create improved *VisUI* widgets instead of regular *Scene2D* actors.

## Examples

Check [gdx-lml-vis-tests](https://github.com/czyzby/gdx-lml/tree/master/examples/gdx-lml-vis-tests) for examples of all tags and macros usage. Check [gdx-lml-vis-websocket](https://github.com/czyzby/gdx-lml/tree/master/examples/gdx-lml-vis-websocket) for a simple (yet practical) mock-up application. [gdx-lml-serialization-tests](https://github.com/czyzby/gdx-lml/tree/master/examples/gdx-websocket-serialization-tests) is a tiny application that uses `gdx-lml-vis` to create GUI.

See on-line demo [here](http://czyzby.github.io/gdx-lml/lml-vis). See application mock-up [here](http://czyzby.github.io/gdx-lml/lml-vis-websocket).

## Dependencies

`gdx-lml-vis` is available through the official project creator tool: `gdx-setup` (in third party extensions). However, its version might not be up to date.

Gradle:
```
        compile "com.github.czyzby:gdx-lml-vis:$libVersion.$gdxVersion"
```
`$libVersion` is the current version of the library, usually following `MAJOR.MINOR` schema. `$gdxVersion` is the LibGDX version used to build (and required by) the library. You can check the current library version [here](http://search.maven.org/#search|ga|1|g%3A%22com.github.czyzby%22) - or you can use the [snapshots](https://oss.sonatype.org/content/repositories/snapshots/com/github/czyzby/).

GWT module:
```
        <inherits name='com.github.czyzby.lml.vis.GdxLmlVis' />
```
On GWT, you might also want to include sources of `gdx-lml-vis` and currently used `VisUI` library version.

## Usage

Start with [LML tutorial](https://github.com/czyzby/gdx-lml/wiki/Tutorial) - *gdx-lml-vis* works pretty much like regular LML, it just parses templates to different actors, provides additional widgets and contains default skins. Make sure to check out the [example project](https://github.com/czyzby/gdx-lml/tree/master/examples/gdx-lml-vis-tests) and [VisUI features](https://github.com/kotcrab/VisEditor/wiki/VisUI).

### Upgrading regular *gdx-lml* to *gdx-lml-vis*

Instead of building your parser with `Lml` class (or `LmlParserBuilder`), use `VisLml` (or `VisLmlParserBuilder`). Everything else is pretty straightforward: `VisLmlSyntax` overrides default *Scene2D* actors with improved *VisUI* widgets, so you can use all of the standard tags. Your existing *LML* templates should work out of the box - you might only need to change style names here and there, since *VisUI* uses its own custom `Skin`.

### Non-*GWT* features

To add non-GWT features (like the `FileChooser` support or various file validators from `FormValidator` class), use `ExtendedVisLml#extend(LmlParser)` method. This will register all the extra attributes and tags at the cost of nasty compilation errors on GWT platform.

### Including *gdx-lml-vis* in [Autumn MVC](https://github.com/czyzby/gdx-lml/tree/master/mvc)

In one of your components (possibly in a configuration component, if you have one), include an annotated field with syntax instance:

```
        @LmlParserSyntax VisLmlSyntax syntax = new VisLmlSyntax();
```

Make sure that you call `VisUI.load()` and register Vis skin in LML parser. This can be easily done with an initiation method:

```
        @Initiate(priority = AutumnActionPriority.TOP_PRIORITY) // ...or custom, higher.
        private void initiate(final SkinService skinService) {
            VisUI.load(); // VisUI.load(Gdx.files.internal("path/to/your/skin.json"));
            skinService.addSkin("default", VisUI.getSkin());
        }
```

By using `SkinService#addSkin` method, the disposing of VisUI skin and registering it in LML parser is already handled for you. Still, you might also want to manually dispose of the `ColorPicker` instance (as it is reused for performance reasons) if you ever used one:

```
        @Destroy
        public static void destroyColorPicker() {
            ColorPickerContainer.dispose();
        }
```

Note that by making the method static, instance of the class containing the method will not be kept, so it can be safely garbage collected after application initiation. This is very useful if you want to keep such settings in a single configuration class.

## What's new

Make sure to check `gdx-lml` changes as well!

1.5 -> 1.6

- Macro marker was changed from `@` to `:`. While it required to switch a single character in the actual source code, this is actually a major update. This change breaks all LML templates that used any macro tags. `@` was a poor choice in the first place: it can be confused with the i18n bundle marker (also `@`) and is not a valid XML character. Since `DTD` supported was added, invalid macro sign was no longer an option. Quick conversion tip: replace `<@` and `</@` with `<:` and `</:` in all `*.lml` files.
- Removed `/*` alias for comment macro. Since `DTD` creator was added to LML, now it is possible to create templates that are somewhat-valid `XML` files. `/*` was the only default tag that used forbidden `XML` characters.
- This version brought a lot of syntax additions to make LML more XML-friendly. Thanks to the new named macro attributes, LML templates can now be valid XML without sacrificing the utility that macros bring. Make sure to go through `gdx-lml` changes for more info.
- `progressBar` and `slider` tags, previously completely non-parental, can parse text between their tags - provided that it's a valid float. The parsed number will be set their initial value. Note that it's *not* a simple `value` attribute alias: `value` attribute is parsed *before* the actor is created, so it cannot trigger any registered change listeners. On the other hand, data between tags is parsed *after* actor is created (and has processed its attributes), so it *can* trigger the listeners.

1.4 -> 1.5

- As `vertical` style was removed from default skin, `Separator` tag no longer supports `vertical`/`horizontal` attributes.
- `VerticalFlowGroup` and `HorizontalFlowGroup` support. To use these groups in a `dragPane` tag, set `type` attribute to `vFlow` or `hFlow` (as always, case ignored).
- `ListView` support. Now you can display a collection of values in a customized way. Note that currently `ListView` does **NOT** work on GWT due to reflection use in VisUI 1.0.1. It will be fixed in future versions.
- Added `deadzoneRadius` attribute to `draggable` tag.

### Archive
Older change logs are available in `CHANGES.md` file.