# Affiliate Form Fix - December 3, 2025

## Problem
The affiliate application form at `/partner` was not working properly:
- Form submissions were not being saved to the database
- No confirmation message was displayed after submission
- Form fields remained visible with user data after clicking submit
- Users had no feedback that their application was submitted

## Root Cause
The WPForms form configuration (post ID 5066) was missing the required `'id'` field in its JSON data structure. This caused:
1. Hundreds of PHP warnings: `Undefined array key "id"` in WPForms core files
2. Form rendering failures
3. Submission processing failures
4. No database entries being created

## Solution

### 1. Fixed Form Configuration
**File:** WordPress database - `wp_posts` table, post ID 5066

Added the missing `'id'` field to the form JSON:
```json
{
  "id": "5066",
  "field_id": "8",
  "fields": { ... },
  "settings": {
    "ajax_submit": "1",
    "confirmations": {
      "1": {
        "id": "1",
        "name": "Default Confirmation",
        "type": "message",
        "message": "<custom success message>",
        "message_scroll": "1"
      }
    }
  }
}
```

### 2. Simplified Affiliate Plugin Form Handler
**File:** `wp-content/plugins/sst-affiliate-manager/includes/class-form-handler.php`

Removed problematic filter hooks that were interfering with WPForms:
- Removed `wpforms_frontend_form_data` filter (was breaking form_data array)
- Kept only `wpforms_process_complete` action for affiliate database entry

**New simplified code:**
```php
class SST_Affiliate_Form_Handler {
    private function __construct() {
        // Only hook into form processing, not form rendering
        add_action('wpforms_process_complete', [$this, 'process_submission'], 10, 4);
    }

    public function process_submission($fields, $entry, $form_data, $entry_id) {
        // Creates affiliate entry in custom database table
        // Sends to Zapier if enabled
    }
}
```

### 3. Success Message Configuration
The success message is now configured directly in the WPForms form settings:

**Message:**
- Purple gradient background (#667eea to #764ba2)
- 🎉 emoji and "Thank You!" heading
- "We will get back to you" text
- Support email: support@sst.nyc

**Behavior:**
- Form fields disappear after submission
- Success message appears via AJAX (no page reload)
- Auto-scrolls to message

### 4. Updated Partner Page Slug
**File:** WordPress database - Page ID 5082

Changed page slug from `/yourmoney` to `/partner` to match production

## Files Changed

### Staging Server
1. `wp-content/plugins/sst-affiliate-manager/includes/class-form-handler.php` - Simplified form handler
2. WordPress database - Post 5066 (WPForms form) - Added 'id' field and success message
3. WordPress database - Post 5082 (Partner page) - Changed slug to 'partner'

## Testing Results
✓ Form loads without PHP errors
✓ Form submissions save to WPForms database
✓ Affiliate entries created in custom affiliate table
✓ Success message displays properly
✓ AJAX submission works correctly
✓ Zapier webhook triggers (if enabled)

## Migration to Production

### Prerequisites
- Production form ID may be different (check with `wp post list --post_type=wpforms`)
- Update form ID in success message if needed

### Steps
1. Copy updated `class-form-handler.php` to production
2. Export form 5066 from staging
3. Update production form with:
   - Add `"id": "<production_form_id>"` field
   - Update success message
   - Ensure `ajax_submit: "1"` is set
4. Verify `/partner` page slug on production
5. Test submission end-to-end

## Rollback Plan
If issues occur on production:
1. Restore previous `class-form-handler.php` from git
2. Revert form configuration via WPForms admin interface
3. Clear WordPress cache

## Coupon Generation

### How It Works
The affiliate plugin automatically generates WooCommerce coupon codes when an affiliate is **approved** (not when they submit the application).

**Process:**
1. User submits affiliate application → Status: "pending"
2. Admin approves affiliate in dashboard → Status: "approved"
3. Plugin creates WooCommerce coupon automatically
4. Affiliate receives email with their unique coupon code and QR code

### Coupon Code Format
- Prefix: `AFFILIATE` (configurable in plugin settings)
- Can include borough-based codes for tracking
- Default: 10% discount
- Individual use only

### If Coupon Deletion
**Q: If we delete the affiliate, does the coupon code also get deleted?**

**A:** This depends on the plugin implementation. Check:
```php
// File: wp-content/plugins/sst-affiliate-manager/includes/class-affiliate.php
// Look for delete() or deactivate() method
```

**Recommended approach:**
- **Deactivate** affiliate instead of deleting (preserves history)
- **Disable** the coupon in WooCommerce (makes it unusable but keeps tracking)
- Only delete if absolutely necessary (may break commission tracking)

### Troubleshooting Coupon Errors
**Error:** "Coupon not generated. WooCommerce may not be active or coupon generation failed"

**Causes:**
1. Affiliate status is "pending" (coupons only generate on "approved")
2. WooCommerce plugin not active
3. WooCommerce API permissions issue
4. Duplicate coupon code exists

**Check:**
```bash
wp plugin list --name=woocommerce
wp wc coupon list --user=admin
```

## Future Improvements
- Add personalized first name to success message (requires dynamic filter)
- Add email confirmation automation
- Implement Zapier webhook for affiliate notifications
- Consider using WPForms admin interface for future form edits to avoid JSON issues
- Add coupon cleanup on affiliate deletion (optional)

## Technical Notes

### Why the 'id' field was critical
WPForms core code expects `$form_data['id']` to exist throughout the rendering and processing lifecycle. Without it:
- Field rendering fails (undefined array key warnings)
- Form validation breaks
- Submission processing can't match the form
- Confirmations don't display

### Why we removed the filter approach
Attempting to modify `form_data` via `wpforms_frontend_form_data` filter was:
- Breaking the array structure
- Causing cascade of undefined index errors
- More complex than needed

The direct database update approach is:
- Simpler and more reliable
- How WPForms expects forms to be configured
- Easier to debug and maintain
