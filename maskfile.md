# Task runner for developing Scdlang syntax highlighting

## run
### run bat (file)
> Print syntect highlighting result

```sh
./scripts/print.sh $file
```

## prepare
> Prepare the ingredients for various tasks 🍳

```sh
mkdir -p dist/rules
mkdir -p dist/syntect/{syntaxes,themes}
docker-compose up --no-start
```

## build
> Please run `prepare build` before running this tasks ⚠

### build vscode
> Generate textmate grammar to be used in VSCode

```sh
run_and_inspect() { $1 && ./scripts/print.sh dist/Scdlang.tmLanguage.json; }

if which dhall-to-json 2>/dev/null; then
  run_and_inspect "dhall-to-json --file syntaxes/Scdlang.dhall --pretty --output dist/Scdlang.tmLanguage.json"
else
  run_and_inspect "docker-compose run --rm --user $(id --user) dhall-json"
fi
```

### build textmate
> Generate textmate grammar in plist format

```sh
[ -f dist/Scdlang.tmLanguage.json ] || mask build vscode >/dev/null
if ./scripts/json2plist.js dist/Scdlang.tmLanguage.json > dist/Scdlang.tmLanguage; then
  ./scripts/print.sh dist/Scdlang.tmLanguage -l xml
fi
```

### build sublime
> Generate sublime-syntax grammar

```sh
[ -f dist/Scdlang.tmLanguage ] || mask build textmate >/dev/null
# [ -f dist/Scdlang.tmLanguage ] || ./scripts/automate-sublime.sh dist/Scdlang.tmLanguage dist/Scdlang.sublime-syntax
if docker-compose run --rm --user $(id --user) convert_syntax; then
  ./scripts/print.sh dist/Scdlang.sublime-syntax -l yaml
fi
```

### build syntect
> Generate dump file for syntect

```sh
# [ "$(ls -A dist/rules)" ] || mask build textmate
linkto() { [ -f $1 ] || ln dist/Scdlang.sublime-syntax $1; }
hardlink() { linkto dist/syntect/syntaxes/Scdlang.sublime-syntax; }
[ -f dist/Scdlang.sublime-syntax ] && hardlink || (mask build sublime >/dev/null; hardlink)

run_and_inspect() { $1 && ./scripts/print.sh examples/one-liner.scl; }

if bat cache --clear 2>/dev/null; then
  run_and_inspect "bat cache --build --source dist/syntect/"
  cp $HOME/.cache/bat/*.bin dist/syntect/
else
  run_and_inspect "docker-compose run --rm --user $(id --user) packdump"
fi
```

### build clear
> Remove all artifacts

```sh
rm dist/syntect/*.bin
rm dist/syntect/*/*
rm dist/Scdlang.*
```

## cleanup
> Cleanup all build tools 🧹

```sh
docker-compose down -v --rmi all --remove-orphans
```