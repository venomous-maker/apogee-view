# apogee-view

Templates for [Apogee](https://github.com/venomous-maker), with Blade's syntax
where it makes sense, plus filters and a few more directives. Templates are
files (`resources/views/users/show.tpl.html` is `"users.show"`), parsed once
into a tree and re-read only when the file changes (`checkForChanges`).

```grav
import "std"
import "apogee-view"

let views = new ViewFactory(["resources/views"])
views.share("appName", "Shop")
views.urlFor = fn(name: string, p: HashMap<string, any>) -> string { return router.url(name, p) }
views.composer("layouts.*", fn(name: string, data: HashMap<string, any>) { data.put("menu", menu()) })
views.helper("money", fn(a: Array<any>, c: RenderContext) -> any { return formatMoney(a[0]) })

return views.response(req, "users.index", {"users": users})
```

```html
@extends('layouts.app')
@section('title', 'Users')
@section('content')
  @forelse(users as user)
    <li @class({"first": loop.first})>{{ loop.iteration }}. {{ user.name | title }}
      @if(user.admin) <b>admin</b> @endif</li>
  @empty
    <p>No users yet.</p>
  @endforelse
  <x-alert type="info"><x-slot:title>Tip</x-slot:title>Invite your team.</x-alert>
@endsection
@push('scripts') <script src="{{ asset('users.js') }}"></script> @endpush
```

## Syntax

- **Output:** `{{ expr }}` (HTML-escaped), `{!! expr !!}` (raw),
  `{{-- comment --}}`. `@{{ ... }}` and `@verbatim ... @endverbatim` print
  braces as they are, and `@@` prints `@`.
- **Conditions:** `@if`/`@elseif`/`@else`/`@endif`, `@unless`, `@isset`,
  `@empty`, `@switch`/`@case`/`@break`/`@default`, `@auth`/`@guest`,
  `@env('local', 'staging')`, `@production`.
- **Loops:** `@foreach(items as item)`, `@foreach(map as key => value)`,
  `@forelse ... @empty ... @endforelse`, and
  `@each('view', items, 'item', 'empty view')`. Inside a loop, `loop` has
  `index`, `iteration`, `remaining`, `count`, `first`, `last`, `even`,
  `odd`, `depth` and `parent`.
- **Layouts:** `@extends('layouts.app')`, `@section('x') ... @endsection`,
  `@section('title', 'Home')`, `@yield('x', 'default')`,
  `@section ... @show` (a layout's default), and `@parent`.
- **Composition:** `@include('view', {"x": 1})`, `@includeIf`,
  `@includeWhen(cond, 'view')`, `@includeFirst(['a', 'b'])`,
  `@push`/`@prepend`/`@stack`, `@once`, `@set(total = a + b)`.
- **Forms:** `@csrf`, `@method('PUT')`, `@error('email') {{ message }} @enderror`,
  `old('email')`, `@checked(cond)`, `@selected`, `@disabled`, `@readonly`,
  `@required`, `@class({"active": on})`, `@json(data)`.
- **Components:** `<x-alert type="error" :count="n">body</x-alert>` renders
  `components/alert`. Its attributes become variables (`:attr` ones are
  expressions; `data-id` becomes `dataId`), along with `attributes` (all of
  them) and `slot` (the body). `<x-slot:title>` or `<x-slot name="title">`
  makes a named slot. `<x-icon name="home" />` is self-closing. A
  component sees only its own variables and the shared ones.

## Expressions

Literals (`"text"`, `'text'`, `42`, `1.5`, `true`, `null`, `[...]`,
`{"key": value}`), variables, `.member`, `[index]` (negative from the end),
`f(args)`, `obj.method(args)`, `+ - * / %`, `~` (concatenation),
`== != < > <= >=`, `&& || !` (and `and`, `or`, `not`), `in` and `not in`,
`a ? b : c`, `a ?? b`, and filters: `x | upper | default("-")` is
`default(upper(x), "-")`.

Maps answer member access by key. Objects answer by field or zero-argument
method, through reflection, so their class needs it: a decorator anywhere
in its hierarchy, as Orbit models have. Strings, lists and maps have
`.length` and `.count`, and lists `.first`/`.last`, `.contains(x)` and
`.join(sep)`.

**Helpers** (each also a filter): `upper`, `lower`, `ucfirst`, `title`,
`trim`, `length`, `count`, `default`, `e`, `json`, `join`, `truncate`,
`nl2br`, `slug`, `plural`, `number(v, decimals)`, `round`, `abs`, `min`,
`max`, `range`, `keys`, `first`, `last`, `str`, `date(v, "%d %b %Y")`,
`now`, `route(name, params)`, `url`, `asset`, `old`, `csrf_token`, `class`.

## With a request

`views.render(name, data, req)` and `views.response(req, name, data)` also
give the template `request`, `user` (the signed-in user), `errors` (the
failed form's, from the session), `old()` and the CSRF token.

## Errors

A `ViewError` names the template and line:
`users.index:12: expected ')' in {{ broken( }}`, `missing @endif`,
`no view 'users.missing' (looked in ...)`.

## Speed

`--release`: a page with a layout, a 100-row loop, an include and stacks
(3.5 KB) renders in about 280 µs. Escaping is one pass straight into the
output.

## Tests

```bash
grav test tests/view.grav
```

MIT licensed.
