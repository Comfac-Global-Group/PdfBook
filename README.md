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
https://github.com/Comfac-Global-Group/PdfBook/issues
