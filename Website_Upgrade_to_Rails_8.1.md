# Rails 6.1 to 8.1 Upgrade Checklist - Gemfile Dependencies

This document lists all gem dependencies that require updates for the Rails 6.1 → 8.1 upgrade.

## Gemfile Dependencies

### 1. Core Rails Gems
- **`rails`** - **🔴 CRITICAL**: Update from `6.1.7.8` to `8.1.x`
  - Update Gemfile: `gem 'rails', '~> 8.1'`
  - Major version jump - requires comprehensive testing
- **`pg`** - **🔴 CRITICAL**: Update from current version (likely `~> 1.1` or `~> 1.2`) to `~> 1.6` or latest `1.6.x`
  - Rails 8.1 recommends pg ~> 1.6 for better performance
  - Update Gemfile: `gem 'pg', '~> 1.6'`
- **`sprockets`** - **🟡 UPDATE**: Update from `>= 4.0` to latest `~> 4.2` or check for `5.x`
  - Update Gemfile: `gem 'sprockets', '~> 4.2'`
- **`sass-rails`** - **🟡 UPDATE**: Review compatibility with Rails 8.1, update to latest `>= 5`
- **`coffee-rails`** - **🔴 DEPRECATED**: Currently `~> 4.2`, consider migrating to JavaScript/TypeScript
  - CoffeeScript support may be removed in future Rails versions
  - Plan migration strategy
- **`uglifier`** - **🔴 REPLACE**: Remove and replace with Terser
  - Rails 7+ uses Terser by default for JS compression
  - Remove: `gem 'uglifier'`
  - Add: `gem 'terser'` (or use built-in)
  - Update `config.assets.js_compressor = :terser` in production.rb

### 2. Server & Infrastructure
- **`puma`** - **🔴 CRITICAL**: Currently `5.0.2` (pinned in production), update to `~> 6.4` or `~> 7.0`
  - Rails 8.1 works best with Puma 6.4+ or 7.x
  - Remove version pin from production group
  - Update Gemfile: `gem 'puma', '~> 6.4'` or `gem 'puma', '~> 7.0'`
  - Significant performance improvements in 6.x/7.x
  - Breaking changes: Minimal, mostly configuration updates

### 3. Asset Pipeline
- **`turbolinks`** - **🔴 CRITICAL REMOVAL**: Currently `~> 5`, **REMOVE** or migrate to Turbo (Hotwire)
  - Turbolinks is deprecated in Rails 7+
  - **Action**: Remove `gem 'turbolinks', '~> 5'`
  - **Replacement**: Add `gem 'turbo-rails'`
  - Update all JavaScript files using `turbolinks:load` events (31+ files)
  - Update views with `data-turbolinks-track` attributes
  - See section 9 in full upgrade checklist for file list
- **`turbo-rails`** - **🟡 ADD**: Add for Rails 8.1 compatibility
  - Update Gemfile: `gem 'turbo-rails'`
  - Ensure Turbo is properly configured
  - Review Turbo 8.x features if available
- **`jquery-rails`** - **🟡 REVIEW**: May need updates or removal depending on usage
- **`rails_12factor`** - **🔴 REMOVE**: Remove from production group
  - Rails 7+ includes these features by default
  - No longer needed

### 4. ActiveJob & Background Jobs
- **`delayed_job_active_record`** - **🟡 UPDATE**: Update to Rails 8.1 compatible version
  - Review ActiveJob changes in Rails 8.1
  - **🔴 BREAKING**: `enqueue_after_transaction_commit` configuration option removed in Rails 8.1
  - Update job configurations to align with new defaults
  - Update `config/initializers/delayed_job_config.rb`
- **`daemons`** - **🟡 REVIEW**: Review compatibility with Rails 8.1

### 5. Authentication & Authorization
- **`devise`** - **🟡 UPDATE**: Update to Rails 8.1 compatible version (>= 4.9.5+)
  - Currently unpinned, check for latest Devise version supporting Rails 8.1
  - Update Gemfile: `gem 'devise', '~> 4.9'` or latest compatible
- **`omniauth`** - **🔴 MAJOR UPDATE**: Currently `1.9.1` (pinned), update to `~> 2.1` or `2.2+`
  - Omniauth 2.x is required for Rails 7.2+ and 8.1
  - Remove pin, update Gemfile: `gem 'omniauth', '~> 2.1'`
  - **Breaking changes**: Major API changes in Omniauth 2.x
  - Must update all omniauth providers simultaneously
- **`omniauth-facebook`** - **🟡 UPDATE**: Update to latest compatible version
  - Update alongside Omniauth 2.x
- **`omniauth-google-oauth2`** - **🟡 UPDATE**: Update to latest compatible version
  - Update alongside Omniauth 2.x
- **`omniauth-linkedin-oauth2`** - **🟡 UPDATE**: Currently `1.0.0`, update to latest compatible version
  - Update alongside Omniauth 2.x
  - Verify Rails 8.1 compatibility

### 6. Admin Interface
- **`activeadmin`** - **🟡 UPDATE**: Update to Rails 8.1 compatible version (>= 3.2.5+)
  - Currently unpinned, check for ActiveAdmin 3.3.x or 4.x if available
  - Update Gemfile: `gem 'activeadmin', '~> 3.2'` or latest
  - Review breaking changes in ActiveAdmin changelog

### 7. File Uploads
- **`carrierwave`** - **🟡 UPDATE**: Update to Rails 8.1 compatible version
  - Currently unpinned, verify compatibility with Rails 8.1
  - Update to latest compatible version
- **`carrierwave_direct`** - **🟡 REVIEW**: Review compatibility with Rails 8.1
- **`mini_magick`** - **🟡 UPDATE**: Update to latest version
- **`carrierwave-imageoptimizer`** - **🟡 REVIEW**: Review compatibility with Rails 8.1
- **`fog-aws`** - **🟡 REVIEW**: Review compatibility, may need updates

### 8. Payment Processing
- **`stripe`** - **🔴 MAJOR UPDATE**: Currently `~> 5.0` (likely `5.55.0`), update to `~> 9.0` or `10.x`
  - Stripe SDK 9.x+ has better Rails 8.1 support
  - Update Gemfile: `gem 'stripe', '~> 9.0'` or latest `10.x`
  - **Breaking changes**: Major API changes between 5.x and 9.x/10.x
  - Requires comprehensive code review of all Stripe integrations
  - Update all services in `app/services/checkout/` directory
  - Test payment processing thoroughly

### 9. Database & Search
- **`acts-as-taggable-on`** - **🔴 MAJOR UPDATE**: Currently likely `10.0.0`, update to `~> 12.0` or latest `12.x`
  - Version 12.x supports Rails 7.1, 7.2, and 8.1
  - Update Gemfile: `gem 'acts-as-taggable-on', '~> 12.0'`
  - **Breaking changes**: May require database migrations
  - Review migration guide before upgrading
  - Test all tagging functionality
- **`pg_search`** - **🟡 UPDATE**: Update to latest compatible version
  - Currently unpinned, update to latest Rails 8.1 compatible version
- **`textacular`** - **🟡 REVIEW**: Review compatibility with Rails 8.1

### 10. Other Gems Requiring Updates
- **`simple_form`** - **🟡 UPDATE**: Update to Rails 8.1 compatible version
  - Currently unpinned, update to latest compatible version
- **`kaminari`** - **🟡 UPDATE**: Update to latest compatible version
  - Currently unpinned, update to latest Rails 8.1 compatible version
- **`friendly_id`** - **🟡 UPDATE**: Update to latest compatible version
  - Currently unpinned, update to latest Rails 8.1 compatible version
- **`draper`** - **🟡 UPDATE**: Update to Rails 8.1 compatible version
  - Currently unpinned, update to latest compatible version
- **`jbuilder`** - **🟡 UPDATE**: Update from `~> 2.5` to latest `~> 2.12` or check for `3.x`
  - Update Gemfile: `gem 'jbuilder', '~> 2.12'`
- **`acts_as_follower`** - **🟡 REVIEW**: Review compatibility (GitHub gem)
  - Currently from GitHub, verify Rails 8.1 compatibility
- **`redis`** - **🟡 UPDATE**: Update to latest compatible version
  - Currently unpinned, update to latest `~> 5.3` or `5.4+`
- **`redis-namespace`** - **🟡 UPDATE**: Update alongside Redis
- **`dalli`** - **🟡 UPDATE**: Update to latest version
  - Rails 7.2+ removed deprecated `Dalli::Client` instances support
  - Review all `Dalli::Client` usage
- **`memcachier`** - **🟡 UPDATE**: Update alongside Dalli
- **`rack-cors`** - **🟡 UPDATE**: Update to latest compatible version
- **`rack-timeout`** - **🟡 UPDATE**: Update to latest compatible version
- **`sitemap_generator`** - **🟡 UPDATE**: Update to latest compatible version
- **`geocoder`** - **🟡 UPDATE**: Update to latest compatible version
- **`rollbar`** - **🟡 UPDATE**: Update to latest compatible version
- **`newrelic_rpm`** - **🟡 UPDATE**: Update to latest compatible version
- **`twilio-ruby`** - **🟡 UPDATE**: Update to latest `~> 6.0` or latest
- **`httparty`** - **🟡 UPDATE**: Update to latest compatible version
- **`rest-client`** - **🟡 UPDATE**: Update to latest compatible version
- **`smarter_csv`** - **🟡 UPDATE**: Update to latest compatible version
- **`maxminddb`** - **🟡 UPDATE**: Update to latest compatible version
- **`timezone`** - **🟡 UPDATE**: Update to latest compatible version
- **`faker`** - **🟡 UPDATE**: Update to latest `~> 3.3` or latest
- **`nio4r`** - **🟡 UPDATE**: Update from `~> 2.5` to latest compatible version

### 11. Testing Gems
- **`rspec-rails`** - **🟡 UPDATE**: Update to Rails 8.1 compatible version (>= 7.0+)
  - Currently unpinned, RSpec 7.x or 8.x may be required
  - Update Gemfile: `gem 'rspec-rails', '~> 7.0'` or latest
- **`rails-controller-testing`** - **🔴 DEPRECATED**: May need removal or replacement
  - Consider removing if not actively used
- **`shoulda-matchers`** - **🟡 UPDATE**: Update to Rails 8.1 compatible version
  - Currently unpinned, update to latest compatible version
- **`database_cleaner`** - **🟡 REVIEW**: Rails 7+ and 8.1 have built-in transaction rollback
  - Consider removing if not needed
  - Update to latest version if keeping
- **`factory_bot_rails`** - **🟡 UPDATE**: Update to latest compatible version
  - Currently unpinned, update to latest compatible version
- **`capybara`** - **🟡 UPDATE**: Update to latest compatible version
  - Currently unpinned, update to latest compatible version
- **`vcr`** - **🟡 UPDATE**: Update to latest compatible version
- **`better_errors`** - **🟡 UPDATE**: Update to latest compatible version
- **`letter_opener`** - **🟡 UPDATE**: Update to latest compatible version
- **`letter_opener_web`** - **🟡 UPDATE**: Update alongside Letter Opener

### 12. Development Gems
- **`rubocop`** - **🟡 UPDATE**: Update to latest version for Rails 8.1 style guide
  - Currently unpinned, update to latest version
  - May need to update `.rubocop.yml` configuration
- **`brakeman`** - **🟡 UPDATE**: Update to latest version for Rails 8.1 security checks
  - Currently unpinned, update to latest version
- **`pry-byebug`** - **🟡 UPDATE**: Update to latest compatible version
- **`listen`** - **🟡 UPDATE**: Update to latest compatible version
- **`foreman`** - **🟡 UPDATE**: Update to latest compatible version

### 13. Gems to Remove
- **`rails_12factor`** - **🔴 REMOVE**: Remove from production group
  - Rails 7+ includes these features by default
- **`uglifier`** - **🔴 REMOVE**: Replace with Terser (see section 1)
- **`turbolinks`** - **🔴 REMOVE**: Replace with turbo-rails (see section 3)
