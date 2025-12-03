# SST.NYC Affiliate Manager

WordPress affiliate program management system with WooCommerce integration.

## Features

- **Affiliate Management**: Approve, deactivate, and manage affiliates
- **Custom Commission Rates**: Set individual commission percentages per affiliate
- **Coupon Management**: Automatic WooCommerce coupon generation with custom discounts
- **Admin UI**: Full WordPress admin interface with status badges and inline editors
- **Form Integration**: WPForms integration for affiliate applications

## Documentation

- [Affiliate Enhancements](./AFFILIATE_ENHANCEMENTS.md) - Feature documentation
- [Code Review](./CODE_REVIEW_AFFILIATE_ENHANCEMENTS.md) - Security and quality review
- [Testing Guide](./TESTING_GUIDE.md) - Complete testing scenarios
- [Form Fix Documentation](./AFFILIATE_FORM_FIX.md) - Form submission fixes

## Deployment

**Production:** https://sst.nyc
**Staging:** https://staging.sst.nyc

### Quick Start

1. **Backup the site:**
   ```bash
   ./backup_website.sh
   ```

2. **View affiliate applications:**
   - Go to WordPress Admin → Affiliates → All Affiliates

3. **Configure settings:**
   - WordPress Admin → Affiliates → Settings
   - Set form ID (Production: 5025, Staging: 5066)

## Backup & Restore

### Create Backup

```bash
./backup_website.sh
```

This creates a timestamped backup in `./backups/` containing:
- Database SQL dump
- wp-content directory (plugins, themes, uploads)
- wp-config.php

### Restore Backup

```bash
# Extract backup
tar -xzf backups/sst_nyc_backup_YYYYMMDD_HHMMSS.tar.gz

# Upload files to server
scp -r files_TIMESTAMP/wp-content user@server:/path/to/wordpress/

# Import database
wp db import database_TIMESTAMP.sql
```

## Database Schema

### Affiliates Table (`zush_sst_affiliates`)

- `affiliate_id` - Unique affiliate ID (e.g., AFF-001)
- `first_name`, `last_name`, `email`, `phone` - Contact info
- `company` - Company/organization name
- `status` - pending, approved, rejected, inactive
- `commission_rate` - Custom commission percentage (0-100)
- `coupon_code` - WooCommerce coupon code
- `referral_link` - Unique referral URL
- `created_at`, `approved_at` - Timestamps

## Security

✅ All inputs sanitized
✅ All outputs escaped
✅ WordPress nonces on all forms
✅ Capability checks (manage_options)
✅ No SQL injection vulnerabilities
✅ No XSS vulnerabilities

See [Code Review](./CODE_REVIEW_AFFILIATE_ENHANCEMENTS.md) for details.

## Version History

### v2.0 (December 3, 2025)
- Added deactivate/reactivate functionality
- Custom commission rates per affiliate
- Custom coupon discounts per affiliate
- Complete admin UI implementation
- Menu reorganization (Partnership before Contact)
- Form ID configuration

### v1.0 (Prior)
- Initial affiliate manager
- Basic approval workflow
- Automatic coupon generation
- WPForms integration

## Support

For questions or issues, see the documentation files or check the error logs:
- Production: `/home/u629344933/domains/sst.nyc/logs/`
- Database: `zush_sst_affiliates` table

## License

Proprietary - SST.NYC

---

🤖 Generated with [Claude Code](https://claude.com/claude-code)
