# PdfBook MediaWiki Extension

A MediaWiki extension that composes books from wiki pages and exports them as PDF documents.

## What This Extension Does

PdfBook allows you to:

- **Export category pages as PDF books** – Automatically compiles all pages within a category into a single PDF document
- **Export linked pages as PDF books** – Creates a PDF from pages linked in a bulleted list on any wiki page
- **Export single pages as PDF** – Convert individual wiki pages to PDF format
- **Customize PDF appearance** – Configure fonts, margins, headers, footers, table of contents, and more
- **Add a "Print as PDF" action tab** – Quick access button for logged-in users (optional)

## Requirements

- **MediaWiki**: Version 1.40.0 or later
- **HTMLDoc**: Must be installed on your server at `/usr/bin/htmldoc` or `/usr/local/bin/htmldoc`
- **PHP**: Compatible with your MediaWiki installation

## Installation

### Step 1: Install HTMLDoc

HTMLDoc is the underlying tool that converts HTML to PDF. Install it on your server:

**Debian/Ubuntu:**
```bash
sudo apt-get update
sudo apt-get install htmldoc
```

**RHEL/CentOS/Fedora:**
```bash
sudo yum install htmldoc
# or
sudo dnf install htmldoc
```

**Verify installation:**
```bash
which htmldoc
# Should return: /usr/bin/htmldoc or /usr/local/bin/htmldoc
```

### Step 2: Download the Extension

Clone or download this repository into your MediaWiki `extensions` directory:

```bash
cd /path/to/mediawiki/extensions
git clone https://github.com/Comfac-Global-Group/PdfBook.git
```

### Step 3: Enable the Extension

Add the following line to your `LocalSettings.php` file:

```php
wfLoadExtension( 'PdfBook' );
```

### Step 4: Configure (Optional)

Add configuration options to `LocalSettings.php` as needed:

```php
# Show a "Print as PDF" tab for logged-in users (default: false)
$wgPdfBookTab = true;

# Force PDF download instead of browser view (default: true)
$wgPdfBookDownload = true;

# Custom margins (optional)
$wgPdfBookLeftMargin = '2cm';
$wgPdfBookRightMargin = '2cm';
$wgPdfBookTopMargin = '1.5cm';
$wgPdfBookBottomMargin = '1.5cm';

# Font settings (optional)
$wgPdfBookFont = 'Arial';
$wgPdfBookFontSize = '10';
$wgPdfBookFontSpacing = 1.5;

# Table of contents levels (default: 2)
$wgPdfBookTocLevels = '3';

# Link color in hex (default: 217A28)
$wgPdfBookLinkColour = '0066CC';

# Page numbering (default: yes)
$wgPdfBookNumbering = 'yes';

# Custom HTMLDoc options (advanced)
$wgPdfBookOptions = '--no-links';
```

## Docker Deployment

For Docker-based MediaWiki deployments, you need a **custom Dockerfile** that includes HTMLDoc and persists the extension configuration.

### Dockerfile

Create a `Dockerfile` in your project:

```dockerfile
# Use the official MediaWiki image as base
FROM mediawiki:1.40

# Install HTMLDoc (required for PdfBook)
RUN apt-get update && apt-get install -y \
    htmldoc \
    && rm -rf /var/lib/apt/lists/*

# Verify HTMLDoc installation
RUN which htmldoc

# Copy PdfBook extension into the container
# Option 1: Clone during build
RUN cd /var/www/html/extensions && \
    git clone https://github.com/Comfac-Global-Group/PdfBook.git

# Option 2: Copy local PdfBook folder (if you have it locally)
# COPY ./PdfBook /var/www/html/extensions/PdfBook

# Ensure proper permissions
RUN chown -R www-data:www-data /var/www/html/extensions/PdfBook
```

### Docker Compose

Create a `docker-compose.yml` file:

```yaml
version: '3.8'

services:
  mediawiki:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: mediawiki-pdfbook
    restart: unless-stopped
    ports:
      - "8080:80"
    volumes:
      # Persist MediaWiki data (images, cache, LocalSettings.php)
      - ./data/images:/var/www/html/images
      - ./data/cache:/var/www/html/cache
      
      # Persist LocalSettings.php (contains PdfBook configuration)
      - ./data/LocalSettings.php:/var/www/html/LocalSettings.php:ro
      
      # Optional: Persist the PdfBook extension for development
      # - ./PdfBook:/var/www/html/extensions/PdfBook
      
    environment:
      - MEDIAWIKI_DB_TYPE=mysql
      - MEDIAWIKI_DB_HOST=db
      - MEDIAWIKI_DB_NAME=mediawiki
      - MEDIAWIKI_DB_USER=wikiuser
      - MEDIAWIKI_DB_PASSWORD=wikipass
    depends_on:
      - db
    networks:
      - wiki-network

  db:
    image: mysql:8.0
    container_name: mediawiki-db
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
      MYSQL_DATABASE: mediawiki
      MYSQL_USER: wikiuser
      MYSQL_PASSWORD: wikipass
    volumes:
      - db_data:/var/lib/mysql
    networks:
      - wiki-network

volumes:
  db_data:

networks:
  wiki-network:
    driver: bridge
```

### LocalSettings.php for Docker

Create `./data/LocalSettings.php` with PdfBook configuration:

```php
<?php
# ... your existing MediaWiki configuration ...

# Enable PdfBook extension
wfLoadExtension( 'PdfBook' );

# PdfBook Configuration
$wgPdfBookTab = true;           # Show "Print as PDF" tab for logged-in users
$wgPdfBookDownload = true;      # Force PDF download

# Margins
$wgPdfBookLeftMargin = '2cm';
$wgPdfBookRightMargin = '2cm';
$wgPdfBookTopMargin = '1.5cm';
$wgPdfBookBottomMargin = '1.5cm';

# Font settings
$wgPdfBookFont = 'Arial';
$wgPdfBookFontSize = '10';
$wgPdfBookFontSpacing = 1.5;

# Other settings
$wgPdfBookTocLevels = '3';
$wgPdfBookLinkColour = '0066CC';
$wgPdfBookNumbering = 'yes';
```

### Deployment Steps

```bash
# 1. Create project directory
mkdir -p mediawiki-pdfbook/data
cd mediawiki-pdfbook

# 2. Create Dockerfile and docker-compose.yml (as shown above)

# 3. Create LocalSettings.php with PdfBook config in ./data/

# 4. Build and start the containers
docker-compose up -d --build

# 5. Verify HTMLDoc is installed
docker exec mediawiki-pdfbook which htmldoc
# Should output: /usr/bin/htmldoc

# 6. Complete MediaWiki setup at http://localhost:8080
# After setup, copy LocalSettings.php to ./data/LocalSettings.php
```

### Ensuring Persistence

The key to persistence is using **Docker volumes**:

| Path in Container | Purpose | Persistence |
|-------------------|---------|-------------|
| `/var/www/html/images` | Uploaded files, PDF cache | ✅ Persisted via volume |
| `/var/www/html/LocalSettings.php` | Extension config | ✅ Persisted via volume |
| `/usr/bin/htmldoc` | HTMLDoc binary | ✅ Built into image |
| `/var/www/html/extensions/PdfBook` | Extension code | ✅ Built into image (or volume mount for dev) |

### Updating PdfBook in Docker

To update the extension:

```bash
# Option 1: Rebuild the image (pulls latest from git)
docker-compose down
docker-compose up -d --build

# Option 2: Update inside running container
docker exec -it mediawiki-pdfbook bash
cd /var/www/html/extensions/PdfBook
git pull
exit
docker restart mediawiki-pdfbook
```

### Troubleshooting Docker Issues

**HTMLDoc not found in container:**
```bash
# Check if HTMLDoc is installed
docker exec mediawiki-pdfbook dpkg -l | grep htmldoc

# If missing, rebuild the image
docker-compose down
docker-compose up -d --build
```

**PDF cache not persisting:**
```bash
# Ensure images directory is writable
docker exec mediawiki-pdfbook chown -R www-data:www-data /var/www/html/images
docker exec mediawiki-pdfbook chmod 755 /var/www/html/images
```

**Permission denied on LocalSettings.php:**
```bash
# Check file permissions on host
chmod 644 ./data/LocalSettings.php
```

## Usage

### Method 1: Export a Category as PDF

1. Navigate to any category page (e.g., `Category:MyBook`)
2. Add `?action=pdfbook` to the URL:
   ```
   https://yourwiki.com/index.php/Category:MyBook?action=pdfbook
   ```
3. The extension will compile all pages in the category into a single PDF

### Method 2: Export Linked Pages as PDF

1. Create a wiki page with a bulleted list of links to other pages:
   ```markdown
   * [[Page One]]
   * [[Page Two]]
   * [[Chapter Three|Page Three]]
   ```
2. Add `?action=pdfbook` to the URL:
   ```
   https://yourwiki.com/index.php/MyBook?action=pdfbook
   ```

### Method 3: Export a Single Page

Add `?action=pdfbook&format=single` to any page URL:
```
https://yourwiki.com/index.php/MyPage?action=pdfbook&format=single
```

### URL Parameters

You can customize the output using URL parameters:

| Parameter | Description | Example |
|-----------|-------------|---------|
| `format=single` | Export single page only | `?action=pdfbook&format=single` |
| `format=html` | Export as HTML instead of PDF | `?action=pdfbook&format=html` |
| `pdfnotitle=true` | Suppress page titles | `?action=pdfbook&pdfnotitle=true` |
| `pdfnothumbs=true` | Use full-size images instead of thumbnails | `?action=pdfbook&pdfnothumbs=true` |
| `pdfNumbering=no` | Disable page numbering | `?action=pdfbook&pdfNumbering=no` |
| `pdffont=Times` | Change font family | `?action=pdfbook&pdffont=Times` |
| `pdffontsize=12` | Change font size | `?action=pdfbook&pdffontsize=12` |
| `pdfLeftMargin=2cm` | Set left margin | `?action=pdfbook&pdfLeftMargin=2cm` |

## Troubleshooting

### Error: "HTMLDoc was not found in your system"

**Cause:** HTMLDoc is not installed or not in the expected location.

**Solution:**
1. Install HTMLDoc (see Step 1 in Installation)
2. Verify it's installed: `which htmldoc`
3. If installed in a custom location, create a symlink:
   ```bash
   sudo ln -s /custom/path/htmldoc /usr/local/bin/htmldoc
   ```

### PDF Generation Fails Silently

**Possible Causes & Solutions:**

1. **Permissions issue:** Ensure the web server user can write to the upload directory:
   ```bash
   sudo chown -R www-data:www-data /path/to/mediawiki/images/
   sudo chmod -R 755 /path/to/mediawiki/images/
   ```

2. **HTMLDoc path issue:** Check that HTMLDoc is accessible by the web server user

3. **Memory limit:** Increase PHP memory limit in `php.ini`:
   ```ini
   memory_limit = 256M
   ```

### Images Not Appearing in PDF

**Cause:** Image paths may not be resolving correctly.

**Solutions:**
- Use `pdfnothumbs=true` parameter to use full-size images
- Check that `$wgUploadPath` and `$wgUploadDirectory` are correctly configured in `LocalSettings.php`
- Ensure image files are readable by the web server

### PDF Looks Different from Web View

**Expected Behavior:** The PDF is a print-optimized view that may differ from the web layout.

**Tips:**
- Elements with class `noprint` are automatically removed from PDF output
- Use CSS classes to hide/show elements specifically for print
- Test with different `pdffont` settings for better rendering

### "Print as PDF" Tab Not Showing

**Cause:** `$wgPdfBookTab` is disabled or user is not logged in.

**Solution:**
```php
# In LocalSettings.php
$wgPdfBookTab = true;  // Enable the tab
```
Note: The tab only appears for logged-in users.

### Large Books Timeout or Fail

**Cause:** PHP execution time limit or memory exhaustion.

**Solutions:**
1. Increase PHP execution time in `php.ini`:
   ```ini
   max_execution_time = 300
   ```
2. Process books in smaller chunks (fewer pages per category)
3. Enable caching by ensuring `$wgUploadDirectory` is writable

### Cache Not Working

**Cause:** Upload directory is not writable.

**Solution:**
```bash
sudo chown www-data:www-data /path/to/mediawiki/images
sudo chmod 755 /path/to/mediawiki/images
```

Cached PDFs are stored as `pdf-book-cache-*` files in the upload directory.

## Advanced Configuration

### Excluding Pages

Use the `pdfExclude` parameter to skip specific pages:
```
?action=pdfbook&pdfExclude=Page One,Page Two
```

Or configure globally in `LocalSettings.php`:
```php
$wgPdfBookExclude = ['Sidebar', 'Footer', 'Navigation'];
```

### Custom HTMLDoc Path

If HTMLDoc is installed in a non-standard location:
```php
$wgPdfBookHtmlDocPath = '/opt/htmldoc/bin/htmldoc';
```

### Logging

PDF exports are logged to the `pdf` log type. View exports in the wiki log:
```
https://yourwiki.com/index.php/Special:Log/pdf
```

## License

This extension is licensed under the GNU General Public License v2.0 or later (GPL-2.0-or-later).

## Credits

- **Original Author:** Aran Dunkley (Organic Design)
- **Contributors:** Igor Absorto, Professional Wiki
- **Documentation:** [Kimi](https://kimi.ai) and [Claude](https://claude.ai) (AI Assistants)
- **Version:** 3.1.0
- **Official Documentation:** https://www.mediawiki.org/wiki/Extension:PdfBook

## Support

For issues and feature requests, please use the GitHub issue tracker:
https://github.com/debtcompliance/PdfBook/issues
