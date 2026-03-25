source "https://rubygems.org"

# Hello! This is the default Gemfile for new Jekyll apps. You may wish to use different
# gems. See the documentation at https://jekyllrb.com/docs/plugins/installation/ for the
# complete list of official plugins and community gems that Jekyll supports.

gem "jekyll", "~> 3.9.0"

# This is the default theme for new Jekyll sites. You may change this to anything you like.
gem "minima", "~> 2.5"

# If you have any plugins, put them here!
group :jekyll_plugins do
  gem "jekyll-feed", "~> 0.12"
  gem "jekyll-seo-tag"
  gem "jekyll-sitemap"
  gem "jekyll-include-cache"
end

# Windows and JRuby does not include zoneinfo files, so bundle the tzinfo-data gem
# and associated library.
platforms :mingw, :x64_mingw, :mswin, :jruby do
  gem "tzinfo", (>= 1, < 3)
  gem "tzinfo-data"
end

# Performance-booster for watching directories on Windows
gem "wdm", "~> 0.1.1", :platforms => [:mingw, :x64_mingw, :mswin]

# Lock `http_parser.rb` to `v0.6.x` on JRuby builds, because earlier versions of this gem
# do not have a pre-compiled extension for all of the Ruby versions supported by Jekyll.
gem "http_parser.rb", "~> 0.6.0", :platforms => [:jruby]
