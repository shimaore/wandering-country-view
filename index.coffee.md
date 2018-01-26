    app_name = 'wandering-country'

This is the version number for the Couch app / design document itself.
Notice that it doesn't contain a patch-level indication (because we
can't do resolution), but the major and minor should follow semver
(minor change = new functionality with no changes to existing
functionalities, major change = changes to existing functionality).

    {version} = require './package.json'
    {major,minor} = require 'semver'

    app_version = "#{major version}.#{minor version}"

Ensure backward-compatibility between patches and minors!

- Do not change the content or the length of existing keys.
- Do not change the set returned for a given query.
- Only extend values (this is only possible if values are objects).

    app = "#{app}-#{app_version}"

Current views
-------------

    {p_fun} = require 'coffeescript-helpers'

    couchapp = (options) ->
      options ?= {}
      extra = ''
      extra = "var normalize_account = #{options.normalize_account};" if options.normalize_account?

      _id: "_design/#{app}"
      version: version
      language: 'javascript'
      views:
        all:
          map: p_fun extra, require('./all') options
          reduce: '_count'

    module.exports = {app,app_version,couchapp}
