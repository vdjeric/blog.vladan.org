source 'https://rubygems.org'

gem 'wdm', '>= 0.1.0' if Gem.win_platform?
gem 'jekyll'

# Always use current version of Github's github-pages pages
require 'json'
require 'open-uri'
versions = JSON.parse(open('https://pages.github.com/versions.json').read)
gem 'github-pages', versions['github-pages']

# Plugins
gem 'jekyll-sitemap'
# Uncomment this plugin once this issue is fixed (very soon): https://github.com/sparklemotion/nokogiri/issues/1256
#gem 'jemoji'

