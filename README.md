# Arcanus

`arcanus` is Ruby-based tool to make working with encrypted secrets in your
git repositories easier.

* [Requirements](#requirements)
* [Installation](#installation)
* [Usage](#usage)
* [API](#api)
* [License](#license)

## Requirements

* Ruby 2.0.0+

## Installation

### Installing Arcanus in a repository for the very first time

If you are introducing Arcanus to a repository, run:

```bash
cd path/to/your/repo
gem install arcanus
arcanus setup
```

The wizard will guide you though the remaining steps.

### Working in a repository with Arcanus already installed

If the repository is already set up to use Arcanus, you simply need to unlock
the protected key with the password that was used to encrypt it when the
original developer ran `arcanus setup`. Ideally this password will be stored in
a shared password manager used by your team.

```bash
cd path/to/your/repo
gem install arcanus
arcanus unlock
```

The wizard will guide you through the unlock process.

## Usage

All commands are of the form `arcanus command`, where `command` is from the list
below. Help documentation will be shown if no command is provided.

* [edit](#edit-key-path-value)
* [export](#export---type)
* [help](#help)
* [setup](#setup)
* [show](#show-key-path)
* [unlock](#unlock)
* [version](#version)

### `edit [key-path] [value]`

Edits the content of the chest with the editor specified by your `$EDITOR`
environment variable.

You can optionally specify a key path (i.e. `some.nested.key.name`) and a value
to modify a single value at a time from the command line without opening an
editor. The key path must already exist, i.e. you cannot use this form to add a
new key.

**Note:** Care should be taken when adding secrets via the command line, since
they may be kept in your shell history. Some shells automatically filter out
commands which are prefixed with a space.

### `export [--type]`

Outputs the decrypted values in a format suitable for consumption by other
programs.

The command optionally allows you to specify the type of format.

#### Default (no flag specified)

By default, `export` will output in a format to make variables accessible in
your local shell, but will not export those values so that they are passed to
other programs invoked by your shell. Thus you could write a `bash` script
like:

**my-script.sh**
```bash
echo ${SOME_SECRET:-not-defined}
```

```bash
eval $(arcanus export)
echo $SOME_SECRET
./my-script.sh
```

**Output**
```
my-secret
not-defined
```

#### `--shell` (export shell variables)

Similar to the default, but export the variables.

**my-script.sh**
```bash
echo ${SOME_SECRET:-not-defined}
```

```bash
eval $(arcanus export --shell)
echo $SOME_SECRET
./my-script
```

**Output**
```
my-secret
my-secret
```

#### `--docker` (docker env-file)

If you are running containers which need environment variables passed to them,
you can use the `--docker` flag to export them in an unescaped format that
Docker will parse.

```bash
docker run --rm --env-file <(arcanus export --docker) -it centos:7.1.1503 env
```

**Output**
```
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
<...truncated...>
SOME_SECRET=my-secret
```

**Note:** We're using the
[process substitution](https://en.wikipedia.org/wiki/Process_substitution)
operator `<(...)` so we don't need to create any temporary files.

### `help`

Display a quick summary of available commands.

### `setup`

Creates an Arcanus chest in the current repository.

This is an interactive wizard that will generate a key and lock it using a
password of your choosing.

### `show [key-path]`

Decrypts and displays the entire contents of the chest.

You can optionally specify a key path (i.e. `some.nested.key.name`) to display
a single key, which is useful in shell scripts that don't need access to _all_
the secrets in the chest.

### `unlock`

Unlocks the key so that the chest can be opened without a password.

This command will be run by other developers who are working with the
repository for the first time. In order to view/edit secrets, they'll need
the original password used during `arcanus setup` so that they can unlock
their local key and not have to enter the password again.

### `version`

Displays the Arcanus version.

## API

Arcanus exposes a simple API for retrieving your secrets in Ruby applications.

Assuming Arcanus is already setup in your repository via
`arcanus setup`/`arcanus unlock`, you can write something like:

```ruby
require 'arcanus'

... = Arcanus.chest['my_secret']
```

## License

This project is released under the [MIT license](LICENSE.md).
