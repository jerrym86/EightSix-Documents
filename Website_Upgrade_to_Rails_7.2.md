## Service Updates for Upgrading From 6.1 to Rails 7.2

### 1. Core Rails Gems
- **`rails`** - Update from `6.1.7.8` to `7.2.x`
- **`pg`** - **🔴 CRITICAL**: Update from current version to `~> 1.5` or latest `1.5.x`
  - Rails 7.2 recommends pg ~> 1.5 for better performance and feature support
  - Update Gemfile: `gem 'pg', '~> 1.5'`
- **`sprockets`** - Update to Rails 7 compatible version (>= 4.0, may need >= 4.1)
- **`sass-rails`** - Review compatibility
- **`coffee-rails`** - Review compatibility (consider migrating to JavaScript)
- **`uglifier`** - May need replacement (Rails 7 uses terser by default)

### 2. Server & Infrastructure
- **`puma`** - **🔴 CRITICAL**: Currently `5.0.2` (pinned in production), update to `~> 6.0` or latest `6.4.0+`
  - Rails 7.2 works best with Puma 6.x
  - Remove version pin from production group
  - Update Gemfile: `gem 'puma', '~> 6.0'`
  - Significant performance improvements in 6.x
  - Breaking changes: Minimal, mostly configuration updates

### 3. Asset Pipeline
- **`turbolinks`** - **🔴 CRITICAL**: Turbolinks is deprecated in Rails 7. Consider:
  - Migrating to Turbo (Hotwire)
  - Or removing Turbolinks entirely
  - Update all JavaScript files using `turbolinks:load` events
  - Update views with `data-turbolinks-track` attributes

### 4. ActiveJob & Background Jobs
- **`delayed_job_active_record`** - Update to Rails 7 compatible version
- **`daemons`** - Review compatibility

### 5. Authentication & Authorization
- **`devise`** - Update to Rails 7 compatible version (>= 4.9)
- **`omniauth`** - Currently `1.9.1`, update to latest compatible version
- **`omniauth-facebook`** - Update for compatibility
- **`omniauth-google-oauth2`** - Update for compatibility
- **`omniauth-linkedin-oauth2`** - Currently `1.0.0`, update if available

### 6. Admin Interface
- **`activeadmin`** - Update to Rails 7 compatible version (>= 2.13)

### 7. File Uploads
- **`carrierwave`** - Update to Rails 7 compatible version
- **`carrierwave_direct`** - Review compatibility
- **`mini_magick`** - Review compatibility
- **`carrierwave-imageoptimizer`** - Review compatibility

### 8. Payment Processing
- **`stripe`** - **🔴 CRITICAL**: Currently `~> 5.0` (likely `5.55.0`), update to `~> 9.0` or latest `9.x`
  - Stripe SDK 9.x has better Rails 7.2 support
  - Update Gemfile: `gem 'stripe', '~> 9.0'`
  - **Breaking changes**: Major API changes between 5.x and 9.x
  - Requires comprehensive code review of all Stripe integrations
  - Update all services in `app/services/checkout/` directory
  - Test payment processing thoroughly

### 9. Other Gems Requiring Updates
- **`simple_form`** - Update to Rails 7 compatible version
- **`kaminari`** - Update to Rails 7 compatible version
- **`friendly_id`** - Update to Rails 7 compatible version
- **`draper`** - Update to Rails 7 compatible version
- **`jbuilder`** - Update from `~> 2.5` to Rails 7 compatible version
- **`acts-as-taggable-on`** - Update for Rails 7 compatibility
- **`acts_as_follower`** - Review compatibility (GitHub gem)
- **`pg_search`** - Update for Rails 7 compatibility
- **`textacular`** - Review compatibility

### 10. Testing Gems
- **`rspec-rails`** - Update to Rails 7 compatible version
- **`rails-controller-testing`** - May be deprecated, review
- **`shoulda-matchers`** - Update for Rails 7
- **`database_cleaner`** - Review (Rails 7 has built-in transaction rollback)
- **`factory_bot_rails`** - Update for Rails 7
- **`capybara`** - Update for Rails 7

### 11. Development Gems
- **`rubocop`** - Update for Rails 7 style guide
- **`brakeman`** - Update for Rails 7 security checks
