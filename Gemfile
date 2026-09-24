source 'https://rubygems.org'

# Pin to whatever GitHub Pages currently runs: https://pages.github.com/versions/
# Update with `bundle update github-pages`.
gem 'github-pages', group: :jekyll_plugins

# No longer bundled with Ruby 3.0+, needed by `jekyll serve`
gem 'webrick'

# Windows and JRuby don't ship zoneinfo files
platforms :mingw, :x64_mingw, :mswin, :jruby do
  gem 'tzinfo', '>= 1', '< 3'
  gem 'tzinfo-data'
end

gem 'wdm', '~> 0.1', platforms: [:mingw, :x64_mingw, :mswin]
