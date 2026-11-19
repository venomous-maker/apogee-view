# Changelog

Every release of apogee-view, newest first. Each is a `vX.Y.Z` tag with a GitHub
release carrying its section; the `grav` line in `grav.toml` says which
compiler versions it builds with.

## Unreleased

The first tagged release. Needs grav 1.7.0 or newer.

Templates for [Apogee](https://github.com/venomous-maker), with Blade's syntax where it makes sense, plus filters and a few more directives. Templates are files (`resources/views/users/show.tpl.html` is `"users.show"`), parsed once into a tree and re-read only when the file changes (`checkForChanges`).
