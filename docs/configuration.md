# Configuration file

A configuration file specifies the details of enumerating and operating on license metadata for apps.

Configuration can be specified in either YML or JSON formats.  Examples below are given in YML.

## Restricting sources

The `sources` configuration property specifies which sources `licensed` will use to enumerate dependencies.
By default, `licensed` will generally try to enumerate dependencies from all sources.  As a result,
the configuration property should be used to explicitly disable sources rather than to enable a particular source.

Be aware that this configuration is separate from an individual sources `#enabled?` method, which determines
whether the source is valid for the current project.  Even if a source is enabled in the configuration
it may still determine that it can't enumerate dependencies for a project.

```yml
sources:
  bower: true
  rubygem: false
```

`licensed` determines which sources will try to enumerate dependencies based on the following rules:
1. If no sources are configured, all sources are enabled
2. If no sources are set to true, any unconfigured sources are enabled
```yml
sources:
  bower: false
  # all other sources are enabled by default since there are no sources set to true
```
3. If any sources are set to true, any unconfigured sources are disabled
```yml
sources:
  bower: true
  # all other sources are disabled by default because a source was set to true
```

## Applications

What is an "app"?  In the context of `licensed`, an app is a combination of a source path and a cache path.

Configuration can be set up for single or multiple applications in the same repo.  There are a number of settings available for each app:
```yml
# If not set, defaults to the directory name of `source_path`
name: 'My application'

# Path is relative to git repository root
# If not set, defaults to '.licenses'
cache_path: 'relative/path/to/cache'

# Path is relative to git repository root and specifies the working directory when enumerating dependencies
# Optional for single app configuration, required when specifying multiple apps
# Defaults to current directory when running `licensed`
source_path: 'relative/path/to/source'

# Sources of metadata
sources:
  bower: true
  rubygem: false

# Dependencies with these licenses are allowed by default.
allowed:
  - mit
  - apache-2.0
  - bsd-2-clause
  - bsd-3-clause
  - cc0-1.0

# These dependencies are explicitly ignored.
ignored:
  rubygem:
    - some-internal-gem

  bower:
    - some-internal-package

# These dependencies have been reviewed.
reviewed:
  rubygem:
    - bcrypt-ruby

  bower:
  - classlist # public domain
  - octicons
```

### Specifying a single app
To specify a single app, either include a single app with `source_path` in the `apps` configuration, or remove the `apps` setting entirely.

If the configuration does not contain an `apps` value, the root configuration will be used as an app definition.  In this scenario, the `source_path` is not a required value and will default to the directory that `licensed` was executed from.

If the configuration contains an `apps` value with a single app configuration, `source_path` must be specified.  Additionally, the applications inherited `cache_path` value will contain the application name.  See [Inherited cache_path values](#inherited_cache_path_values)

### Specifying multiple apps
The configuration file can specify multiple source paths to enumerate metadata, each with their own configuration.

Nearly all configuration settings can be inherited from root configuration to app configuration.  Only `source_path` is required to define an app.

Here are some examples:

#### Inheriting configuration
```yml
sources:
  go: true
  rubygem: false

ignored:
  rubygem:
    - some-internal-gem

reviewed:
  rubygem:
    - bcrypt-ruby

cache_path: 'path/to/cache'
apps:
  - source_path: 'path/to/app1'
  - source_path: 'path/to/app2'
    sources:
      rubygem: true
      go: false
```

In this example, two apps have been declared.  The first app, with `source_path` `path/to/app1`, inherits all configuration settings from the root configuration.  The second app, with `source_path` `path/to/app2`, overrides the `sources` configuration and inherits all other settings.

#### Default app names
An app will not inherit a name set from the root configuration.  If not provided, the `name` value will default to the directory name from `source_path`.
```yml
apps:
  - source_path: 'path/to/app1'
  - source_path: 'path/to/app2'
```

In this example, the apps have names of `app1` and `app2`, respectively.

#### Inherited cache_path values
When an app inherits a `cache_path` from the root configuration, it will automatically append it's name to the end of the path to separate it's metadata from other apps.  To force multiple apps to use the same path to cached metadata, explicitly set the `cache_path` value for each app.
```yml
cache_path: 'path/to/cache'
apps:
  - source_path: 'path/to/app1'
    name: 'app1'
  - source_path: 'path/to/app2'
    name: 'app2'
  - source_path: 'path/to/app3'
    name: 'app3'
    cache_path: 'path/to/app3/cache'
```

In this example `app1` and `app2` have `cache_path` values of `path/to/cache/app1` and `path/to/cache/app2`, respectively.  `app3` has an explicit path set to `path/to/app3/cache`

```yml
apps:
  - source_path: 'path/to/app1'
```

In this example, the root configuration will contain a default cache path of `.licenses`.  `app1` will inherit this value and append it's name, resulting in a cache path of `.licenses/app1`.

## Source specific configuration

See the [source documentation](./sources) for details on any source specific configuration.
