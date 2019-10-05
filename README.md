# FastIgnore

[![travis](https://travis-ci.org/robotdana/fast_ignore.svg?branch=master)](https://travis-ci.org/robotdana/fast_ignore)

Filter a directory tree using a .gitignore file. Recognises all of the [gitignore rules](https://www.git-scm.com/docs/gitignore#_pattern_format) ([except one](#known-issues))

```ruby
FastIgnore.new(relative: true).sort == `git ls-files`.split("\n").sort
```

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'fast_ignore'
```

And then execute:
```sh
$ bundle
```
Or install it yourself as:
```sh
$ gem install fast_ignore
```

## Usage

```ruby
FastIgnore.new.each { |file| puts "#{file} is not ignored by the .gitignore file" }
```

Like many other enumerables, `FastIgnore#each` can return an enumerator

```ruby
FastIgnore.new.each.with_index { |file, index| puts "#{file}#{index}" }
```

By default, FastIgnore will return full paths. To return paths relative to the current working directory, use:

```ruby
FastIgnore.new(relative: true).to_a
```

You can specify other gitignore-style files to ignore as well. These rules will be appended after the gitignore file in order (order matters for negations)
```ruby
FastIgnore.new(ignore_files: '/absolute/path/to/my/ignore/file').to_a
FastIgnore.new(ignore_files: ['/absolute/path/to/my/ignore/file', '/and/another']).to_a
```
You can also supply an array of rule strings. These rules will be appended after the gitignore and any other files in order (order matters for negations)
```ruby
FastIgnore.new(ignore_rules: '.DS_Store').to_a
FastIgnore.new(ignore_rules: ['.git', '.gitkeep']).to_a
```

To use only another ignore file or an array of rules, and not try to load a gitignore file:
```ruby
FastIgnore.new(ignore_files: '/absolute/path/to/my/ignore/file', gitignore: false)
FastIgnore.new(ignore_rules: %w{my*rule /and/another !rule}, gitignore: false)
```

By default, FastIgnore will look in the directory the script is run in (`PWD`) for a gitignore file. If it's somewhere else:
```ruby
FastIgnore.new(gitignore: '/absolute/path/to/.gitignore').to_a
```
Note that the location of the .gitignore file will affect rules beginning with `/` or ending in `/**`

To check if a single file is allowed, use
```ruby
FastIgnore.new.allowed?('relative/path')
FastIgnore.new.allowed?('./relative/path')
FastIgnore.new.allowed?('/absolute/path')
FastIgnore.new.allowed?('~/home/path')
```

### Using an includes list.

Building on the gitignore format, FastIgnore also accepts a list of allowed or included files.

```
# a line like this means any files named foo will be included
# as well as any files within directories named foo
foo
# a line beginning with a slash will be anything in a directory that is a child of the PWD
/foo
# a line ending in a slash will will include any files in any directories named foo
# but not any files named foo
foo/
# negated rules are slightly different from gitignore
# in that they're evaluated after all the other the matching files rather than
# in sequence with other rules
fo*
!foe
# otherwise this format deals with *'s and ?'s and etc as you'd expect from gitignore.
```

These can be passed either as files or as an array or string rules
```ruby
FastIgnore.new(include_files: '/absolute/path/to/my/include/file', gitignore: false)
FastIgnore.new(include_rules: %w{my*rule /and/another !rule}, gitignore: false)
```

The string array rules additionally expects to handle all kinds of `ARGV` values, so will correctly resolve absolute paths and paths beginning with `~`, `../` and `./`. Note: it will *not* resolve e.g. `/../` in the middle of a rule that doesn't begin with any of `~`,`../`,`./`,`/`. This is not available for the file form.

## Known issues
- Doesn't take into account project excludes in `.git/info/exclude`
- Doesn't take into account globally ignored files in `git config core.excludesFile`.
- Doesn't follow this rule in the gitignore documentation because I don't understand what it means that isn't covered by other rules:

  > [If the pattern does not contain a slash /, Git treats it as a shell glob pattern and checks for a match against the pathname relative to the location of the `.gitignore` file (relative to the toplevel of the work tree if not from a `.gitignore` file)](https://www.git-scm.com/docs/gitignore#_pattern_format)

  if someone can explain it with examples [make an issue please](https://github.com/robotdana/fast_ignore/issues/new)

## Development

After checking out the repo, run `bin/setup` to install dependencies. Then, run `rake spec` to run the tests. You can also run `bin/console` for an interactive prompt that will allow you to experiment.

To install this gem onto your local machine, run `bundle exec rake install`. To release a new version, update the version number in `version.rb`, and then run `bundle exec rake release`, which will create a git tag for the version, push git commits and tags, and push the `.gem` file to [rubygems.org](https://rubygems.org).

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/robotdana/fast_ignore.

## License

The gem is available as open source under the terms of the [MIT License](https://opensource.org/licenses/MIT).
