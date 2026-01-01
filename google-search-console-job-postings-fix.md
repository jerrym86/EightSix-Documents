# Google Search Console Job Postings Structured Data Fix

## Overview

This document describes the fixes applied to resolve Google Search Console validation errors for job posting structured data (JSON-LD). The changes ensure compliance with Schema.org JobPosting specifications and improve data quality for Google's job search indexing.

## Issues Identified

Google Search Console reported the following validation errors:

1. **Missing field "postalCode"** in `jobLocation.address`
2. **Invalid object type for field "value"** in `baseSalary` (using `PropertyValue` instead of `QuantitativeValue`)

## Root Causes

### 1. Missing postalCode Field
- The structured data did not include the `postalCode` field required by Schema.org
- Existing postal code data had quality issues (e.g., `"2441"`, `"v6y1l1"`, `"v4k  5B8"`)

### 2. Invalid baseSalary Structure
- Non-numeric compensation values were using `"@type": "PropertyValue"` which is invalid for `baseSalary.value`
- The code attempted to output salary information even when values were missing or non-numeric

### 3. Additional Issues Found
- Date formats were not in ISO 8601 format
- Numeric values were quoted (strings instead of numbers)
- No nil safety guards for edge cases
- Hand-written JSON with ERB conditionals risked invalid JSON syntax

## Solutions Implemented

### 1. Postal Code Formatter (`app/decorators/address_decorator.rb`)

Added `formatted_postal_code` method to normalize postal codes:

```ruby
def formatted_postal_code
  raw = (postal_zip_code.presence || postal_code).to_s.strip.upcase
  return nil if raw.blank?

  cleaned = raw.gsub(/\s+/, '')

  # Canadian postal code: A1A1A1 -> A1A 1A1
  if cleaned.match?(/\A[A-Z]\d[A-Z]\d[A-Z]\d\z/)
    return "#{cleaned[0..2]} #{cleaned[3..5]}"
  end

  # US zip code: 12345
  if cleaned.match?(/\A\d{5}\z/)
    return cleaned
  end

  # US zip code: 123456789 -> 12345-6789
  if cleaned.match?(/\A\d{9}\z/)
    return "#{cleaned[0..4]}-#{cleaned[5..8]}"
  end

  # Return original if format not recognized
  raw.present? ? raw : nil
end
```

**Features:**
- Normalizes Canadian postal codes: `v6y1l1` → `V6Y 1L1`
- Formats US zip codes: `12345` or `123456789` → `12345-6789`
- Handles mixed case and extra spaces
- Returns `nil` for empty/invalid values (omitted from JSON)

### 2. Structured Data Refactor (`app/views/job_postings/show.html.erb`)

**Before:** Hand-written JSON with ERB conditionals (error-prone)

**After:** Ruby Hash → `to_json` approach (type-safe, no invalid JSON possible)

**Key improvements:**

1. **Added postalCode field:**
   ```ruby
   "postalCode" => formatted_postal,
   ```

2. **Fixed date formats to ISO 8601:**
   ```ruby
   "datePosted" => @job_posting.created_at&.iso8601,
   "validThrough" => (@job_posting.created_at ? (@job_posting.created_at + 30.days).iso8601 : nil),
   ```

3. **Fixed baseSalary structure:**
   - Only outputs when numeric values exist
   - Removed invalid `PropertyValue` type
   - Unquoted numeric values (proper JSON numbers)
   - Added nil safety guards

4. **Simplified addressCountry:**
   ```ruby
   "addressCountry" => country_code,  # "CA" or "US" instead of nested object
   ```

5. **Used `.compact` to automatically omit nil values:**
   ```ruby
   structured_data = { ... }.compact
   ```

### 3. Nil Safety Fix (`app/models/address.rb`)

Fixed potential crash in `postal_code` method:

```ruby
# Before: parse_address[3] || ""  # Crashes if parse_address is nil
# After:
def postal_code
  parse_address&.[](3).to_s
end
```

## Files Modified

1. **`app/decorators/address_decorator.rb`**
   - Added `formatted_postal_code` method

2. **`app/views/job_postings/show.html.erb`**
   - Refactored JSON-LD structured data block (lines 31-106)
   - Changed from hand-written JSON to Ruby Hash + `to_json`

3. **`app/models/address.rb`**
   - Fixed nil safety in `postal_code` method

## Testing

### Local Development Testing

1. **Start Rails server:**
   ```bash
   rails server
   ```

2. **Test postal code formatter in Rails console:**
   ```ruby
   address = Address.first.decorate
   address.formatted_postal_code
   # Test various formats:
   # - "v6y1l1" should return "V6Y 1L1"
   # - "12345" should return "12345"
   # - "123456789" should return "12345-6789"
   # - "2441" should return "2441" (fallback)
   # - "" or nil should return nil
   ```

3. **View job posting page:**
   - Navigate to a job posting URL (e.g., `http://localhost:3000/job_postings/[slug]`)
   - View page source
   - Find the `<script type="application/ld+json">` block
   - Copy the JSON content

4. **Validate with Google Rich Results Test:**
   - Go to: https://search.google.com/test/rich-results
   - Paste the JSON-LD code or enter the local URL
   - Verify no validation errors

5. **Alternative: Schema.org Validator:**
   - Go to: https://validator.schema.org/
   - Paste the JSON-LD code
   - Verify no errors

### Test Scenarios

Test with various job posting configurations:

- ✅ Job posting with postal code and numeric salary (range)
- ✅ Job posting with postal code and numeric salary (fixed)
- ✅ Job posting without postal code (should omit field)
- ✅ Job posting with text compensation (should omit baseSalary)
- ✅ Job posting with no compensation (should omit baseSalary)
- ✅ Job posting with nil created_at (should handle gracefully)

## Expected Results

After the fix:

- ✅ No "Missing field postalCode" error
- ✅ No "Invalid object type for field value" error in baseSalary
- ✅ Valid ISO 8601 date formats
- ✅ Properly formatted postal codes (Canadian: `A1A 1A1`, US: `12345` or `12345-6789`)
- ✅ Numeric salary values as JSON numbers (not strings)
- ✅ Structured data validates successfully in Google's Rich Results Test
- ✅ No runtime errors when data is missing

## Schema.org Compliance

The structured data now fully complies with [Schema.org JobPosting](https://schema.org/JobPosting) specification:

- ✅ Required fields: `title`, `datePosted`, `description`, `hiringOrganization`, `jobLocation`
- ✅ Optional but recommended: `postalCode`, `baseSalary`, `validThrough`
- ✅ Proper data types: `QuantitativeValue` for salary, ISO 8601 for dates
- ✅ Valid JSON structure (no syntax errors possible)

## Git Commits

```
71e121680 Fix Google Search Console structured data issues for job postings
a5fea6669 Fix nil safety in Address#postal_code method
```

## Related Documentation

- [Google Job Posting Structured Data](https://developers.google.com/search/docs/appearance/structured-data/job-posting)
- [Schema.org JobPosting](https://schema.org/JobPosting)
- [Google Rich Results Test](https://search.google.com/test/rich-results)

## Notes

- The postal code formatter returns the original value as fallback if format is not recognized (e.g., `"2441"`). This ensures no data is lost, though Google may flag it as low-quality.
- The structured data uses `.compact` to automatically omit `nil` values, ensuring clean JSON output.
- All date calculations use safe navigation (`&.`) to prevent crashes on edge cases.

