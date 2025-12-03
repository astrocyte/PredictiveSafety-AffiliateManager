# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a WordPress LMS management system for SST.NYC, designed to automate maintenance tasks and provide a robust testing environment. The infrastructure includes:

- **Production Site:** SST.NYC on Hostinger
- **Tech Stack:** WordPress + LearnDash (LMS) + WooCommerce + Elementor + Mailchimp
- **Planned Components:** MCP server, testing/staging environment, automation scripts

## Agent Role

Claude Code acts as an agentic SEO expert, LMS administrator, and e-commerce manager for this WordPress site, with capabilities to:
- Optimize content and site structure for search engines
- Manage WordPress configuration, plugins, and themes
- **Create and edit LearnDash courses, lessons, and quizzes**
- **Manage student enrollments and groups**
- **Create and manage WooCommerce products linked to courses**
- **Handle pricing, coupons, and sales reporting**
- Automate maintenance and monitoring tasks
- Provide SEO analysis and recommendations
- Work with Elementor page builder for content layout and design
- Convert and optimize images for web performance

## Specialized Agents

**CRITICAL:** Always use specialized agents for these tasks. Never use wp-cli directly.

### Quick Reference
- **LearnDash** (courses/lessons/enrollments) → `learndash-specialist`
- **Content Writing** (marketing/copy/emails) → `sst-copywriter`

### learndash-specialist
Use for ALL LearnDash operations:
- Course/lesson/quiz management
- Student enrollments and groups
- Course builder modifications
- Fixing course structure issues

### sst-copywriter
Use for ALL content writing:
- Course descriptions and marketing copy
- Email content and website copy
- Requires SST.NYC brand voice (Bill Burr meets Alan Watts meets NYC construction site)

## Architecture (Planned)

### MCP Server Setup
- **Stack:** Linux (Ubuntu/CentOS) + Nginx + MariaDB/Percona + PHP-FPM
- **Caching Layer:** Redis/Memcached for object cache, FastCGI/page cache
- **Security:** UFW/firewalld, Fail2Ban, Wordfence/Sucuri, Let's Encrypt SSL
- **Monitoring:** Prometheus + Grafana for system metrics
- **Backups:** Automated to S3/Google Cloud Storage

### Testing Server
- Mirrors production environment exactly
- Automated data pulls from production (with anonymization)
- Isolated environment using subdomain (test.sst.nyc or staging.sst.nyc)
- Email interception via MailHog/Mailtrap
- Git-based deployment workflow: local → testing → production

## MCP Servers

This project uses the following MCP servers for automation:

### 1. Zapier MCP (Remote)
- **Purpose:** Connect Claude to 8,000+ apps for workflow automation
- **Configuration:** `.claude/mcp_config.json` (not committed - see `.claude/mcp_config.example.json`)
- **Documentation:** `wordpress-mcp-server/ZAPIER_MCP_SETUP.md`
- **Use Cases:** Google Sheets integration, email automation, affiliate program workflows
- **Setup:** Get your MCP URL at https://zapier.com/mcp

### 2. WordPress MCP Server (Local)
- **Purpose:** Direct WordPress/LearnDash management via wp-cli
- **Location:** `wordpress-mcp-server/`
- **See:** Full documentation below

## Development Workflow

When code is added to this repository, it should follow this pattern:
1. Develop locally
2. Test on staging server
3. Deploy to production after approval

## MCP Server (wordpress-mcp-server/)

A hybrid MCP server combining wp-cli and WordPress REST API for site management and SEO optimization.

### Quick Start
```bash
cd wordpress-mcp-server
pip install -e .
cp .env.example .env
# Configure .env with your credentials
python src/server.py
```

See `wordpress-mcp-server/SETUP.md` for complete setup instructions.

### Architecture
- **wp-cli wrapper** (`wp_cli.py`): SSH-based WordPress operations using battle-tested wp-cli
- **REST API client** (`wp_api.py`): Real-time content queries via WordPress REST API
- **SEO analyzer** (`seo_tools.py`): Meta analysis, Elementor content extraction, recommendations
- **Image optimizer** (`image_optimizer.py`): WebP conversion, compression, alt text validation
- **LearnDash manager** (`learndash_manager.py`): Course/lesson/quiz creation and student management
- **WooCommerce manager** (`woocommerce_manager.py`): Product/order/coupon management and sales reporting
- **MCP server** (`server.py`): Exposes 27 tools for complete WordPress management

### Available Tools (27 total)
- Site info: `wp_get_info`, `wp_plugin_list`, `wp_theme_list`
- Content: `wp_post_list`, `wp_get_post`, `wp_search`
- SEO: `seo_analyze_post`, `elementor_extract_content`
- Images: `image_analyze`, `image_optimize`, `image_audit_site`
- LearnDash: `ld_create_course`, `ld_update_course`, `ld_list_courses`, `ld_create_lesson`, `ld_update_lesson`, `ld_create_quiz`, `ld_add_quiz_question`, `ld_enroll_user`, `ld_create_group`
- WooCommerce: `wc_create_product`, `wc_update_product`, `wc_list_products`, `wc_list_orders`, `wc_create_coupon`, `wc_get_sales_report`
- Maintenance: `wp_check_updates`

### Key Design Decisions
- **Maintainability first**: Wraps wp-cli rather than reimplementing WordPress logic
- **Hybrid approach**: Uses wp-cli for operations, REST API for data queries
- **SSH-based**: Secure remote execution without exposing WordPress directly

## SEO Best Practices

### Image Optimization
The MCP server includes comprehensive image optimization tools:
- **Analyze images**: Check format, size, dimensions, alt tags
- **WebP conversion**: Automatic conversion from JPEG/PNG to WebP (30% savings)
- **Smart compression**: Reduce file size while preserving quality
- **Bulk audits**: Scan entire site for optimization opportunities
- **Alt text validation**: Ensure accessibility and SEO compliance

Use `image_audit_site` to identify optimization opportunities across your entire site.

## Key Considerations

- **WordPress URL Configuration:** Always update `site_url` and `home_url` when moving between environments
- **Database Sync:** Implement anonymization for sensitive user data during testing server pulls
- **Email Safety:** Ensure staging environment never sends emails to real users
- **Environment Isolation:** Testing and production must remain completely separate
- **Elementor Data:** Stored as JSON in `_elementor_data` post meta - requires custom parsing
- **JWT tokens expire:** If authentication is rejected, remind user to get a new key

## Server Access

### SSH Connection
- **SSH Host:** 147.93.88.8
- **SSH Port:** 65002
- **SSH Username:** u629344933
- **SSH Password:** RvALk23Zgdyw4Zn
- **Connection Command:** `sshpass -p 'RvALk23Zgdyw4Zn' ssh -p 65002 -o StrictHostKeyChecking=no u629344933@147.93.88.8`

### Directory Structure
- **Production Site:** `/home/u629344933/domains/sst.nyc/public_html/`
- **Staging Site:** `/home/u629344933/domains/sst.nyc/public_html/staging/`
  - Staging is a subdirectory of production, accessible at https://staging.sst.nyc
  - Use `--allow-root` flag for wp-cli commands on staging
  - All wp-cli commands must use full path: `cd /home/u629344933/domains/sst.nyc/public_html/staging && wp ...`