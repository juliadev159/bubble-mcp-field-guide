# Elements and design

Read this when building or changing pages, elements and styles.

Status column: **Observed** = done through the MCP on 2026-10-17 or 2026-10-26 (rows
and sections that mix dates carry the date next to the fact); **Docs** = a tool exists in
the Bubble docs but was not exercised in a test. For Docs rows, build in small steps and
ask the user to look in the editor. No row has the Docs status after the 2026-10-26 test.

| Element or topic | What happened through the MCP | Limits and how to check | Status |
|---|---|---|---|
| Page (responsive, column layout) | Created with heading, button, repeating group and text in one `apply_changes` batch, no issues | Check layout in the editor | Observed |
| Text | Dynamic text worked | If the source field disappears (e.g. after an API re-initialization), `check_app_issues` reports that dynamic data must be printable as text | Observed |
| Button | Style applied; click event scheduling an API workflow worked | Bubble added a default icon to the button by itself; remove it if unwanted | Observed |
| Repeating group | Source: a search with constraints and sort; empty state as an element visible when the count is 0 | Results are filtered by privacy rules for real users, not for the agent | Observed |
| Page workflow | Page-loaded event redirecting users who are not logged in (a guard) | Test as a logged-out user | Observed |
| Styles | The agent assigned a style to each element and set preview text for the canvas (`editor_preview_text`) | Assigning a heading style overrode an explicitly set HTML tag; a warning came back | Observed |
| Padding and margin | Padding lives in the style, margin on the element. Setting padding on the element overrode the style in the editor. On 2026-10-26 the catalog listed padding for groups, inputs and popups, and a padding written on a group matched the editor | 2026-10-17: button padding was not listed in the property catalog, so the write was blind; the user confirmed it in the editor. 2026-10-26: `get_node` does not show padding at all (group or reusable definition), so padding is confirmed only in the editor. A min width shown with fit-to-content is Bubble's default, not something the agent set | Observed |
| Discovering properties | `list_element_types`, then `get_element_type_fields`. On 2026-10-26 the catalog was complete by name for groups, inputs, dropdowns, multiline inputs, checkboxes and popups (tables: see the sub-elements below). A deliberately wrong message name in `validate_expression` returns the list of valid messages for that element | Outside the catalog: reusable instance and definition properties (other names, such as `min_height_px`, seen only in the refusal text); table sub-elements `TableMainAxis`, `TableCrossAxis` and `TableCell` (not in `list_element_types`, but `get_element_type_fields` works for them). The caption of `button_disabled` in the catalog is outdated: the editor calls it "Make clickable" with the inverse meaning. Padding: see above | Observed |
| Catalogs of actions and events | `list_action_types` returns a very large response; the backend event catalog did not fit in one response | Request once, narrowly | Observed |
| Reusable elements | 2026-10-26: `create_reusable`, then an instance with `create_element` whose `type` is the definition id. The instance is a reference, not a copy: edits to the definition show up at once | The instance takes only a short property list: no alignment, min/max width or fit in the parent. `min_height_css` is copied at creation and does not follow the definition. Check the height in the editor (see below) | Observed 2026-10-26 |
| Inputs and forms | 2026-10-26: inputs, dropdown, multiline input and checkbox written from the catalog names; Create a new thing, then `ResetInputs`; the record was saved with all fields on the first submit. With `mandatory` fields left empty, the workflow did not run and the empty fields were highlighted | The palette sets a fixed width even when `horiz_alignment: stretch` is sent. Check the saved record with `search_things` after a submit from the preview | Observed 2026-10-26 |
| Popups | 2026-10-26: `create_element` with `type: Popup` and the page as parent; shown and hidden with `ShowElement` and `HideElement`; listed under Overlays in the editor | No separate tool. The palette sets fit width and min 296 by 296. There is no `is_visible`: a popup is hidden until shown | Observed 2026-10-26 |
| Custom states | 2026-10-26: `create_custom_state`, `update_custom_state` (default changed and changed back), `SetCustomState` from workflows | The key comes back from the create call (`tab_`) and is used as `custom.tab_`, not the display name | Observed 2026-10-26 |
| Conditionals beyond visibility | 2026-10-26: colour, background, text, clickability, border and hover conditionals written and verified; order controlled with `reorder_conditionals` | `container_layout` and other properties marked `noStates` are refused. A style's conditional can leave opacity stuck. Check in the preview, not in read-back (see below) | Observed, with limits 2026-10-26 |
| Tables | 2026-10-26: `create_element` with `type: Table` builds columns, a header row and a repeating row by itself; the data source is an ordinary search | The "current row" in a cell is `ElementAncestor` with `ancestor_type: "TableCrossAxis"` and `fetch_data: true`. Commit `data_source` before the cell expressions. The canvas shows raw expressions without preview text (see below) | Observed, with limits 2026-10-26 |
| Mobile views | 2026-10-26: `create_mobile_view` was refused: "the app has no mobile app", while `get_mobile_views` returned the template's home view | Creating a mobile view and its properties were not tested. Needs a project with a mobile app | Observed, refused 2026-10-26 |

## Working rules

- Read back what you wrote (`get_node`) and compare it with what you intended.
- Anything visual that is not in the property catalog needs the user's eyes in the
  editor. Say which element and which property to look at.
- Keep styles: prefer assigning an existing style over per-element overrides, and
  mention every override you set, since it will win over the style.
- Pass the style in `create_element` and write deviations from it (padding and so on)
  in the next operation. A style in the same create replaces the properties passed with
  it; the MCP returns a warning naming them (Observed 2026-10-26).
- Choose a style by its properties (`get_node` on the style id), not by its name. A theme
  style named for a dark background gave white text on a transparent button, invisible on
  a white page. `style_properties` in the reply to `assign_style` or `create_element`
  describes the previous style, not the new one (Observed 2026-10-26).
- After each create in a row or column, read back whether the palette set a fixed width or
  a min height, and undo it if you did not want it (Observed 2026-10-26: fixed 250 px on
  inputs, 180 px on a checkbox, min 296 by 296 on a popup and a table).
- Write expressions that depend on a container's data type only after the container's
  `data_source` is committed in an earlier call. In the same batch the expression is
  rejected as if there were no source (Observed 2026-10-26).

## Layout of groups

Observed 2026-10-26 on a column group holding a row group with two texts. Everything
below was written through the MCP and then matched in the editor.

- **What is written correctly.** `container_layout` (row or column), the gap
  (`use_gap` with `row_gap` and `column_gap`), `padding_*`, min and max width
  (`min_width_css`, `max_width_css`), fixed or fit width (`single_width`, `fit_width`) and
  alignment. The older fixed-layout fields (`fixed_width`, a percentage min width,
  `use_max_width`) sit next to them in the catalog and are easy to mix up.
- **Read-back is partial.** `get_node` shows margin, alignment, min and max, and gap, but
  not padding. It also shows only the fields that make sense for the element's current
  parent: a text in a row did not list `fit_height`, although the write response gave
  `old_value: true`. Padding is confirmed only in the editor.
- **Vertical alignment in a row.** `container_vert_alignment` on a row is accepted without
  a warning and does nothing; the catalog lists only `container_horiz_alignment` and the
  gap for a row's container properties. Set `vert_alignment` on each child instead.
- **`move_element` resets parent-dependent fields.** After moving a text out of a row and
  back, `order` was right but `vert_alignment` was back to the default (flex-start). Set
  it again after every move.
- **Style versus padding.** A style passed in the same `create_element` replaced the
  padding passed with it. Writing padding in a separate `set_element_properties` call
  worked and shows in the editor as a plain value over the style.
- **Narrow screens.** A row wrapped its second item onto a new line at 375 px with no
  horizontal scroll. A new page has no side padding, so a card goes edge to edge.

## Reusable elements, popups, forms

- **Reusable instance.** The instance has its own short property list. `horiz_alignment`
  was refused on create and on `set_element_properties`; the refusal rolled back the whole
  `apply_changes` batch (Observed 2026-10-26). Alignment and stretch of an instance in its
  parent are not writable through the MCP. `get_node` on the instance shows `custom_id`
  and no children, and the layout of the definition.
- **Definition property names.** The definition uses other names (`min_height_px`,
  `max_width_px`) and refuses `min_height_css`. They are not in `get_element_type_fields`;
  only the refusal text lists them. The definition's `get_node` did not show padding or
  `container_vert_alignment`, although the writes were accepted.
- **Stale min height.** The instance kept `min_height_css: 120px` copied from the
  definition at creation. After the definition was set to `min_height_px: 0`, the
  instance stayed at 120 px, and the property is not writable on the instance. The user
  saw the header taller than its content in the editor. Create the instance only after the
  definition's size is final, or check the height in the editor.
- **Popup.** `create_element` with `type: Popup` and the page as parent; in
  `list_element_types` it has `goesOnDocument: true`. Catalog properties include
  `greyout_color`, `greyout_blur` and `prevent_user_from_closing_through_esc`. A tempId
  from the same batch worked as the `element_id` of `ShowElement` and `HideElement`.
  The popup style brought its own padding and a darkened background. In the preview the
  popup opened at the top centre, not the screen centre; this was left as is.
- **Form fields.** `content_format: email`, `mandatory`, `placeholder`, `choices`
  separated by `\n` and `label` all worked on the first try. A checkbox value is `get_data`
  ("is checked"), a dropdown value is text. `ResetInputs` comes from the workflows guide
  and is not in `list_action_types` for a reusable. Width: the palette set
  `single_width: true` with `min_width_css: 250px` even with `horiz_alignment: stretch`;
  `single_width: false` and `min_width_css: 0px` fixed it, and the MCP added
  `fit_width: false` by itself. A fixed 180 px checkbox wrapped its label to two lines.
- **Names.** Elements keep default names (Input A and so on): `rename_node` is not one of
  the operation types allowed in `apply_changes`.
- **Check the data.** A submit from the preview created one record with all five fields;
  `search_things` confirmed it. The creator was a temporary logged-out user.

## Conditionals

Observed 2026-10-26.

- **What worked.** Background and text colour, text, clickability, border width and hover
  text.
- **Order.** The lower conditional wins. `reorder_conditionals` changed the order, the
  read-back showed it, and the preview and the editor both followed it: hovering a
  disabled button kept its disabled text. Inherited style conditionals show in the editor
  collapsed as "inherited from Style" above the element's own.
- **Theme styles already carry conditionals.** Inputs have invalid, hovered and focused
  states; buttons have not-clickable (opacity 25) and hover. Do not duplicate them on the
  element. They are visible in the `old_states` of the `create_element` reply.
- **Stuck opacity.** After `button_disabled` was cleared, the button was clickable and its
  text normal, but the inline opacity stayed at 0.25 from the style's not-clickable state.
  Setting `opacity: 100` on the element did not help; an explicit conditional on the
  inverse state ("checkbox is checked: opacity 100"), placed last, did. Neither read-back
  nor `check_app_issues` reports this; it showed in the preview DOM.
- **Negation.** The value of a checkbox has no `is_false`; the refused batch of 17
  operations rolled back whole. Use the element message `get_not_data`. A wrong message
  name in `validate_expression` returns the full list (`get_data`, `get_not_data`,
  `isnt_clickable`, `is_valid`, `isnt_valid`, `is_hovered`, `is_focused`, `is_visible`).
- **Refused: `container_layout`.** "property container_layout cannot be used in a
  conditional state", followed by the list of properties that are allowed (padding, margin,
  min and max width, alignment, gap, `is_visible`, opacity, colours, borders). This matches
  the `responsive` guide. Alignment and row gap can change by condition; to switch a row to
  a column, use two groups with visibility by page width, or let the row wrap.
- **Caption.** The read-back of a `button_disabled` conditional shows "Make clickable",
  as the editor does (toggle off); the catalog entry still says "This element isn't
  clickable". The stored value is right; only the catalog caption is out of date.
- **Custom state.** The key came back as `tab_`. Read: `GetElement` on the page, then
  `custom.tab_`. Write: `SetCustomState` with `custom_state: "custom.tab_"`. Changing the
  default with `update_custom_state` and back showed in the preview as the hidden and shown
  groups swapping.

## Tables

Observed 2026-10-26.

- **Structure.** `create_element` with `type: Table` creates three columns
  (`TableMainAxis`), a header row and a repeating row (`TableCrossAxis`, the second with
  `cross_axis_repeat: true`), and a `TableCell` (a row container) in each cell. The Table's
  own catalog has no column or row properties, even with `sections: all`, because they are
  child elements. `get_element_type_fields` works for `TableCrossAxis` (for example
  `make_sticky`, `fixed_number_repeating_axis`, `fixed_number_repeating_axis_count`).
  The repeating row starts with `fixed_number_repeating_axis: true` and a count of 3;
  it was set to false, and with one record the preview showed one row. Adding and
  removing columns was not tried.
- **Content.** Texts go straight into a cell: `parentId` is the `TableCell` id.
- **Current row.** `ElementParent` from a text in a cell was refused: the parent is a cell
  "which does not have a type of data". What worked is `ElementAncestor` with
  `ancestor_type: "TableCrossAxis"` and `fetch_data: true`, which reads as "Current row's
  <type>'s <field>". Without `fetch_data` it returns the row's messages but no fields. The
  `expressions` guide and the tool descriptions only show `ElementParent` for a repeating
  group; this was found by trying values in `validate_expression`.
- **Order of calls.** Data source and cell expressions in one batch: the expressions were
  refused as if there were no source, and the batch rolled back. The source in its own call, then the
  expressions, passed.
- **Palette.** The table came with fit width and min 296 by 296 (set to fill and 0); the
  repeating row has a min height of 77 px, which makes rows tall.
- **Canvas.** The MCP warns that dynamic text needs `editor_preview_text`. Without it the
  editor canvas shows raw expressions ("Current row's ... name"), and a date was cut off by
  the row height. The preview showed the right headers and one row.
- **Not seen.** The empty state of the table with no records for the user.

## Mobile views

Observed 2026-10-26: `create_mobile_view` returned "the app has no mobile app". In the
same app `get_mobile_views` returned one view (the template's home view, a page of
393 by 852, fixed width, column, empty). So the MCP lists a mobile view and refuses to
create another. Creating a mobile view and its properties are untested: a project with a
mobile app is needed. The existing view was not touched.

## What a new app already contains

`create_app` produced an app from a responsive template: index, 404 and password-reset
pages, a mobile view for the home page, about seventy styles and the API Connector
plugin already installed (Observed). So "no styles" and "install the API Connector" are
not the first steps; read the outline and the style list before creating anything of
your own, and prefer an existing style.

## Editing an element's look

What worked through the MCP on an existing button (Observed): text, sizes, visibility,
style assignment, and `padding_*` and `margin_*` on the element.

- **Padding versus margin.** Padding is the inside space and lives in the element's
  style; margin is the outside space and is a property of the element in the layout
  tab. A request such as "more space on the left of this button" can mean either, so
  say which one you changed.
- **Element value over style value.** Setting padding on the element won out over the
  style's padding in the editor (Observed). Do not assume the style wins.
- **Per-side values.** After margin was set on one side, the editor showed the margin
  as "Mixed" and expanded it per side. That is display, not a bug.
- **Blind writes.** The property catalog did not list button padding, yet the write was
  accepted without a warning and took effect, which only the user's screenshot of the
  editor confirmed. An accepted write without a warning is not a confirmation for
  anything visual.

## Test and probe elements

A button that runs a backend workflow, or a list that shows records without a filter,
is a useful test tool, and also something a client may see by accident.

- Name it so the purpose is visible in the editor (the test button was labelled with
  "(test)" in its text).
- Put every such element in the "Left in the app" section of the hand-off report, with
  the page it is on.
- Remove it when the check is done, unless the user wants to keep it.

## Not listed here on purpose

Pagination controls, input validation and similar patterns are Bubble practice rather
than MCP behaviour. This skill only covers what happened through the MCP or what the
docs list; for the rest, build small and have the user look.
