source 'https://rubygems.org'

# Mirror the versions GitHub Pages builds with: https://pages.github.com/versions/
# We don't use the github-pages meta-gem because it pins jekyll-remote-theme,
# which drags in an old, vulnerable rubyzip that this site never uses.
gem 'jekyll', '3.10.0'
gem 'kramdown', '2.4.0'
gem 'kramdown-parser-gfm', '1.1.0'
gem 'rouge', '3.30.0'
gem 'jekyll-sass-converter', '1.5.2'

group :jekyll_plugins do
  gem 'jekyll-sitemap', '1.4.0'
end

# No longer bundled with newer Rubies: webrick (3.0+) for `jekyll serve`,
# base64 and bigdecimal (3.4+) for safe_yaml and liquid
gem 'webrick'
gem 'base64'
gem 'bigdecimal'

# Windows and JRuby don't ship zoneinfo files
platforms :mingw, :x64_mingw, :mswin, :jruby do
  gem 'tzinfo', '>= 1', '< 3'
  gem 'tzinfo-data'
end

gem 'wdm', '~> 0.1', platforms: [:mingw, :x64_mingw, :mswin]
