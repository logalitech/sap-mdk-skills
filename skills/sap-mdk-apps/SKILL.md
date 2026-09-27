---
name: sap-mdk-apps
description: SAP Mobile Development Kit (MDK) guidelines for building metadata driven mobile and web apps with SAP Mobile Services. Use this skill when creating, extending, reviewing or debugging an MDK project, including pages, actions, rules, services, bindings and formatters, offline OData and sync, styling, localization and deployment. Also use it when a page validates but does not render, a rule cannot find a control, a value does not repaint, or a style breaks the bundle. Triggers on mentions of MDK, Mobile Development Kit, SAP Mobile Services, `.page`, `.action`, `.service`, `.global` or `Application.app` files, `_Type`, `{Property}` bindings, `$(L,...)` or `$(C,...)` formatters, `#Page:` target paths, and ClientAPI calls such as `getControl`, `redraw` or `setValue`.
---

# SAP Mobile Development Kit Apps

Guidance for building SAP Mobile Development Kit (MDK) apps: how a project fits together, the syntax it uses, and the rules that prevent common failures.

## When to Use This Skill

- Creating or extending an MDK project: pages, actions, rules, services
- Reading or writing bindings, formatters and target paths
- Writing rules against the ClientAPI
- Building offline apps: defining requests, upload and download
- Styling and localizing an app
- Validating an app, deploying it to SAP Mobile Services and updating devices
- Diagnosing a page that validates but renders nothing, a value that does not repaint, or a failed
  bundle

## Core Principles

1. **Metadata driven**: JSON files describe pages, actions and services, and JavaScript rules carry
   the logic metadata cannot express.
2. **Interpreted at runtime**: a client interprets the metadata, so a change reaches devices as a new
   revision from SAP Mobile Services rather than a new store release.
3. **One project, several runtimes**: the same metadata runs on iOS, Android and the web.
4. **Offline by default on mobile**: services use an offline store unless configured otherwise, and
   the app syncs through its own upload and download actions.
5. **The client decides what renders**: the schema ships with the editor extension and the renderer
   with the client, and nothing checks one against the other.

## Runtimes

| Runtime | Delivered through | Differences |
|---|---|---|
| iOS and Android client | SAP Mobile Services | offline store, native modules in `clientAPI.nativescript`, every navigation shell |
| Web | Cloud Foundry | always online, offline settings ignored, no `nativescript`, no `BottomNavigation` or `SideDrawer` |

## Quick Reference

### Project Layout

| Path | Holds |
|---|---|
| `Application.app` | the entry point: `MainPage`, `Styles`, `Localization`, `_SchemaVersion` and the lifecycle events |
| `Pages/*.page` | one screen each |
| `Actions/*.action` | one declarative operation each: navigate, show a message, create an entity, sync |
| `Rules/**/*.js` | JavaScript modules, one default export each |
| `Services/*.service` | an OData service, with its downloaded `$metadata` beside it as `.<Name>.xml` |
| `Globals/*.global` | constants shared across the app |
| `i18n/*.properties` | UI text, one file per language |
| `Styles/Styles.less` | colours and fonts, applied by class name |
| `Images/`, `Extensions/` | assets, and custom controls |
| `.typings/IClientAPI.d.ts` | what the ClientAPI offers at the project's schema version |

Files reference each other by absolute path, prefixed with the application's `_Name` from
`Application.app`: `/<AppName>/Actions/Save.action`. Pages, controls and actions state their kind
in `_Type` (`Page`, `Control.Type.SectionedTable`, `Action.Type.Navigation`). Controls also carry
a `_Name`, which is how rules and target paths address them.

### How the Pieces Connect

- A page holds a `Control.Type.SectionedTable` whose `Sections` carry the content: object tables,
  cards, key value rows, form cells. The navigation shells are `Control.Type.Tabs`,
  `Control.Type.BottomNavigation` and `Control.Type.SideDrawer`.
- A section reads data through a `Target` (a service, an entity set and query options) and binds
  each field with `{Property}`.
- An event property such as `OnPress`, `OnValueChange` or `OnLoaded` points at an action or a rule.
- An action performs one operation and hands on through `OnSuccess` and `OnFailure`. A sequence is
  a chain of actions.
- A rule receives a `clientAPI`, returns a value or a promise, and runs actions with
  `clientAPI.executeAction('/<AppName>/Actions/X.action')`. Rules can import one another.
- `Application.app` events: `OnLaunch`, `OnResume`, `OnSuspend`, `OnExit`, `OnUnCaughtError`, and
  around a metadata update, `OnWillUpdate` (runs on the old definitions) and `OnDidUpdate` (runs on
  the new ones; a failure there reverts to the previous definitions).

### Syntax in Metadata

| Form | Resolves to |
|---|---|
| `{Price}` | a property of the current binding |
| `/<AppName>/Rules/Total.js` as a value | what that rule returns |
| `$(L,key)` | text from `i18n`, in the current language |
| `$(C,{Price},{CurrencyCode})` | a currency amount. Also `$(N,…)` number, `$(P,…)` percentage, `$(S,…)` scientific |
| `$(D,{Date})` | a date. Also `$(T,…)` time, `$(DT,…)` date and time |
| `$(PLT,ios,android,web)` | a different value per platform |
| `$(DV,target,default)` | the target's value, falling back to the default |
| `#Page:<Name>/#Control:<Name>/#Value` | a control's current value. `#Page:-Current` is the page on screen |
| `#ActionResults:<name>/error` | the error of an earlier action that set `ActionResult._Name` |
| `#Application/#AppData/UserId` | the signed in user |

In a rule the same formatting is `clientAPI.formatCurrency`, `formatDate`, `formatNumber` and the rest, and a target path is read with `clientAPI.evaluateTargetPath()`. Raw OData decimals and timestamps look broken on screen, and these helpers are the way to format them, not hand written code.

### From Change to Device

1. `validate` checks the metadata against `_SchemaVersion`. It never compiles `Styles.less`.
2. Deploy bundles the project, which compiles the styles, uploads it to Mobile Services and reports the new revision number.
3. The client checks for a new revision when the app returns to the foreground, or when an
   `ApplicationUpdate` action runs. `OnWillUpdate` runs before the update is applied; in a project generated from SAP's templates, that event is what shows the `New Version Available!` prompt.
4. Only the device proves the change: a page can validate, bundle and deploy, then render nothing on a given client.

## Implementation Checklist

When changing an MDK app:

- [ ] Each service has its `.<Name>.xml` metadata cache, refreshed from a live session
- [ ] Every rule bound in metadata has a default export
- [ ] No asynchronous rule inside an action's `Properties`
- [ ] Values formatted with the `$(…)` formatters or the `clientAPI.format…` helpers
- [ ] Native calls guarded for the web runtime
- [ ] Every new control or section type checked on the target client
- [ ] `Styles.less` free of emoji and checked with a bundle, since `validate` never compiles it
- [ ] The writes of a document sent on its own share one upload category
- [ ] Logout checked against the app's passcode policy

## Key Guidelines

### Rules and the ClientAPI

- A rule bound in metadata needs a default export. Module level variables persist for the app
  session.
- Compute a value that needs an asynchronous rule, such as a `read()` or a `count()`, before the
  action runs, in a rule that then executes the action. The schema does not say whether an action's
  `Properties` wait for a rule's promise. A control's `Value` may be asynchronous.
- On a `SectionedTable` page, reach a form cell through the table,
  `pageProxy.getControl('<Table>').getControl('Field')`, or read
  `#Page:-Current/#Control:Field/#Value` with `evaluateTargetPath()`. `pageProxy.getControl('Field')`
  returns `undefined` there, without an error.
- An action's `OnSuccess` rule receives an `IClientAPI`, which has no `getPageProxy()`. Navigate from
  `OnSuccess` with a plain `Action.Type.Navigation`.
- `setValue(value, notify)` fires the control's `OnValueChange` unless `notify` is `false`, so a
  handler that sets its own control passes `false`. A date picker's `setValue()` takes a `Date`, not
  a `YYYY-MM-DD` string. To seed a control when a page opens, bind its `Value` rather than calling
  `setValue()` from `OnLoaded`.
- To re-evaluate the rules inside sections, redraw the `SectionedTable` control; a section proxy has
  no `redraw()`. To repaint a FormCell label whose text comes from a rule, call `reset()` on it:
  `redraw()` re-runs the rule without updating the screen.
- Feature-detect `reset()`, which some typings versions omit, and reset only controls on the page on
  screen: on iOS an off-screen control comes back blank.
- A rule bound inside a section gets a `clientAPI` without `nativescript`. Read native state in a
  page rule such as `OnLoaded` or `OnResume`, keep it in a module, and let section rules read it.
- `clientAPI.read()` resolves to an ObservableArray: read it with `result.length` and
  `result.getItem(i)`, because `Array.isArray(result)` is `false` and `result[0]` is `undefined`.
- `executeAction` can resolve for a failed request, and `status` is `0` either way, so check
  `error`.

### Pages, Styling and Text

- Layout comes from the choice of control. No control has margin, padding, a border or a corner
  radius. Sizes exist in a few places only: `Height` on Image, Calendar and Extension sections, and
  `Width` on Image sections and some bar items. Section headers and footers can switch their
  padding off with `UseTopPadding` and `UseBottomPadding`. `Styles.less` sets colours and fonts.
- An empty or unsupported section blanks the whole `SectionedTable` rather than being skipped. An
  untried section type goes in on its own, with a known good state to return to.
- A page with a `Control.Type.Tabs` also needs a `SectionedTable`. A page whose only control is the
  Tabs control validates and crashes at runtime.
- By-type and by-name selectors in `Styles.less`, including the examples in the generated file,
  apply to web controls only. On iOS and Android, style a control through a class name in its
  `Styles` property.
- Only FormCell controls have a `Background` sub-control, so a button with a fill colour has to be
  a Button FormCell. A ButtonTable button's style reaches its label only.
- Keep emoji out of `Styles.less`, comments included. The U+FE0F variation selector breaks the
  bundle with an `Unclosed comment` error reported against the generated `.nss` file.
- An app that switches language at runtime needs an `i18n_en.properties` beside `i18n.properties`:
  `setLanguage('en')` loads the former and does not fall back to the latter.

### Offline and Services

- In the editor, a service needs both files: without the `.<Name>.xml` cache, every binding is
  reported invalid.
- `DefiningRequests` on the `Initialize` action set which data the device holds. A defining request
  added there does not reach a store that already exists; resetting the store picks it up.
  `OfflineOData.Download` also accepts `DefiningRequests`, and its schema does not say whether that
  adds one to an existing store.
- A sync is the app's own `Upload` and `Download` actions, and SAP recommends uploading first.
- To send one document on its own, give each of its writes the same `RequestOptions.UploadCategory`
  and pass that category in the `Upload` action's `UploadCategories`. Only matching writes are sent.
- In an offline app every list reads the local store, so a list that loads says nothing about the
  backend. Only uploads and downloads use the network.
- The `ErrorArchive` holds the requests the backend refused. A transport failure leaves it empty, so
  an empty archive is no proof that an upload went through.

### Tooling and Deploy

- The `.gitignore` in SAP's project templates ignores nothing: each pattern has a comment on the
  same line, and git reads that text as part of the pattern. Move the comments to their own lines.
- `I18nKeyNotFound` in the editor can come from a stale key index. Reload the window before acting
  on it.
- A string literal ending in `.app` inside a rule fails `validate` with `Can not find the
  application`, and the reported line and column do not point at it.
- Refresh service metadata only on a live Cloud Foundry session. On an expired one the refresh can
  write an HTML login page into `Services/.<Name>.xml`, so check that file before committing it.
- Comments in rules ship in cleartext inside the deployed `bundle.js`. Keep internal notes out of
  the source.
- To load a new revision on a device, send the app to the background and reopen it. A resume does
  not always trigger the update check; the user menu's Check for Updates always does. Force-stopping
  the app can end the login session.
- `Action.Type.Logout` with `SkipReset: true` does nothing unless the app has a passcode policy in
  Mobile Services, and SAP's templates generate it with `SkipReset: true`.

## Where to Look

| Need                                        | Source                                                                                                                                                           |
| ------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| a control's properties, any schema question | the `mdk-docs` tool of SAP's MDK MCP server (`@sap/mdk-mcp-server`), or the MDK schemadoc on help.sap.com, whose "Latest" can be newer than the project's schema |
| what a proxy offers in a rule               | the project's `.typings/IClientAPI.d.ts`                                                                                                                         |
| a working ClientAPI call to copy            | the rules SAP's templates generate in `Rules/Application/` and `Rules/Logging/`                                                                                  |

---

When a control property or a schema detail is uncertain, check it with `mdk-docs` or the project's
typings before writing it.
