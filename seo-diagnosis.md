# SEO Technical Troubleshooting: Website Not Indexing

## Scenario
"A new Workcity brand website is not indexing even after sitemap submission."

## Comprehensive Diagnostic Process

### 1. Crawlability Tests

#### A. Manual Crawl Test
**Action**: Check if Googlebot can access the site
- Use Google Search Console → URL Inspection Tool
- Enter homepage URL
- Click "Test Live URL"
- Review results for:
  - HTTP status code (should be 200)
  - Page resources loading
  - JavaScript rendering
  - Any blocked resources

**Common Issues**:
- 500 Server errors
- Timeout issues
- DNS problems
- IP blocking

#### B. Crawl Simulation
**Tools to use**:
- Screaming Frog SEO Spider
- Sitebulb
- OnCrawl

**Steps**:
1. Download and install Screaming Frog
2. Enter website URL
3. Configure spider to act like Googlebot
4. Run crawl and check:
   - Can it discover all pages?
   - Any 404 errors?
   - Server response codes
   - Redirect chains

#### C. Server Response Check
```bash
curl -I https://www.workcityafrica.com
```
Look for:
- Status: 200 OK
- Server response time < 200ms
- Proper headers

### 2. Canonical Checks

#### A. Self-Referencing Canonicals
**Issue**: Pages should have canonical tags pointing to themselves

**Check**:
```html
<!-- In page <head> -->
<link rel="canonical" href="https://www.workcityafrica.com/page-url" />
```

**Common Problems**:
- Canonical pointing to homepage from all pages
- Canonical pointing to staging/dev domain
- Incorrect canonical URLs (missing slash, wrong protocol)

#### B. Canonical Chain Conflicts
**Issues to identify**:
- Page A canonical → Page B
- Page B canonical → Page A
- Canonical pointing to 404 page
- Canonical pointing to redirect

**How to check**:
1. View page source
2. Search for "canonical"
3. Visit canonical URL
4. Confirm it returns 200 and is the intended page

#### C. Tool-Based Check
Use Screaming Frog:
- Crawl site
- Filter: Canonical
- Check "Canonical" column
- Identify any mismatches between URL and canonical

### 3. Robots.txt & No-Index Audit

#### A. Robots.txt Analysis
**Access**: https://www.workcityafrica.com/robots.txt

**Red flags**:
```
User-agent: *
Disallow: /  # BLOCKS EVERYTHING - CRITICAL ISSUE

User-agent: Googlebot
Disallow: /  # Blocks Google specifically
```

**Proper robots.txt example**:
```
User-agent: *
Disallow: /wp-admin/
Disallow: /wp-includes/
Allow: /wp-admin/admin-ajax.php

Sitemap: https://www.workcityafrica.com/sitemap.xml
```

**Testing**:
- Google Search Console → robots.txt Tester
- Test important URLs
- Ensure they're not blocked

#### B. Meta Robots Tag Audit
**Check in page <head>**:
```html
<!-- BAD - Prevents indexing -->
<meta name="robots" content="noindex, nofollow">
<meta name="robots" content="noindex">
<meta name="googlebot" content="noindex">

<!-- GOOD - Allows indexing -->
<meta name="robots" content="index, follow">
<!-- Or omit entirely (default is index, follow) -->
```

**How to check**:
1. View source of key pages
2. Search for "robots"
3. Confirm no noindex directives

**WordPress-specific check**:
- Settings → Reading
- Ensure "Discourage search engines" is UNCHECKED

#### C. HTTP Header Check
**Check X-Robots-Tag header**:
```bash
curl -I https://www.workcityafrica.com | grep -i x-robots-tag
```

Should return nothing or:
```
X-Robots-Tag: index, follow
```

NOT:
```
X-Robots-Tag: noindex
```

### 4. Sitemap Structure Issues

#### A. Sitemap Accessibility
**Basic checks**:
- Visit https://www.workcityafrica.com/sitemap.xml
- Should return XML file, not 404
- Should be accessible without authentication

#### B. Sitemap Format Validation
**Requirements**:
- Proper XML format
- UTF-8 encoding
- Maximum 50,000 URLs per sitemap
- Maximum 50MB uncompressed
- URLs must be properly escaped

**Validation tool**:
- Use https://www.xml-sitemaps.com/validate-xml-sitemap.html
- Google Search Console will also show errors

#### C. Sitemap Content Issues
**Red flags**:
```xml
<!-- Wrong protocol -->
<url>
  <loc>http://www.workcityafrica.com/page</loc>  <!-- Should be https -->
</url>

<!-- Blocked by robots.txt -->
<url>
  <loc>https://www.workcityafrica.com/admin/page</loc>
</url>

<!-- Returns 404 -->
<url>
  <loc>https://www.workcityafrica.com/deleted-page</loc>
</url>

<!-- Canonical points elsewhere -->
<url>
  <loc>https://www.workcityafrica.com/page?utm_source=test</loc>
</url>
```

**Best practices**:
- Only include indexable URLs (200 status)
- Only canonical URLs
- Only URLs you want indexed
- Include lastmod dates
- Use proper priority values (0.0-1.0)

#### D. Multiple Sitemaps Structure
**For large sites, use sitemap index**:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<sitemapindex xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <sitemap>
    <loc>https://www.workcityafrica.com/post-sitemap.xml</loc>
  </sitemap>
  <sitemap>
    <loc>https://www.workcityafrica.com/page-sitemap.xml</loc>
  </sitemap>
</sitemapindex>
```

### 5. Page Speed Indexing Blockers

#### A. Server Response Time
**Critical metrics**:
- TTFB (Time to First Byte) < 200ms
- Page load time < 3 seconds
- Mobile page load < 2 seconds

**How to check**:
- Google PageSpeed Insights
- GTmetrix
- WebPageTest

**Common blockers**:
- Slow server/hosting
- No caching
- Unoptimized images
- Too many HTTP requests
- Render-blocking resources

#### B. JavaScript Rendering Issues
**Problem**: If content is loaded via JavaScript, Google might not see it

**Check**:
1. View page source (Ctrl+U)
2. Search for main content text
3. If not found, it's JavaScript-rendered

**Solution**:
- Ensure critical content in HTML
- Use server-side rendering (SSR)
- Implement dynamic rendering for bots
- Test with Mobile-Friendly Test tool

#### C. Core Web Vitals
**Check**:
- Google Search Console → Core Web Vitals report
- PageSpeed Insights

**Critical thresholds**:
- LCP (Largest Contentful Paint) < 2.5s
- FID (First Input Delay) < 100ms
- CLS (Cumulative Layout Shift) < 0.1

**If failing**:
- Optimize images (WebP format, lazy loading)
- Minify CSS/JS
- Enable compression
- Use CDN
- Reduce server response time

### 6. Search Console Debugging Steps

#### Step 1: Verify Ownership
- Ensure site is verified in Google Search Console
- Check both www and non-www versions
- Verify HTTPS version

#### Step 2: Submit Sitemap
- Sitemaps → Add new sitemap
- Enter: sitemap.xml
- Click Submit
- Wait 24-48 hours

#### Step 3: Request Indexing
- URL Inspection → Enter URL
- Click "Request Indexing"
- Do this for 5-10 key pages

#### Step 4: Check Coverage Report
- Coverage → Review errors
- Common errors:
  - "Discovered - currently not indexed" (low priority)
  - "Crawled - currently not indexed" (quality issue)
  - "Excluded by 'noindex' tag" (remove noindex)
  - "Blocked by robots.txt" (fix robots.txt)

#### Step 5: Check Manual Actions
- Manual Actions → Check for penalties
- If present, follow Google's instructions to resolve

#### Step 6: Monitor Index Status
- Use: `site:workcityafrica.com` in Google
- Check number of indexed pages over time
- Takes 2-4 weeks for new sites typically

### 7. Additional Checks

#### A. Domain Authority Issues
**New domain problems**:
- Brand new domains take longer (sandbox effect)
- No backlinks = low trust
- No brand signals

**Solutions**:
- Build high-quality backlinks
- Create Google Business Profile
- Get listed in directories
- Generate social media presence
- Be patient (3-6 months for new domains)

#### B. Content Quality
**Google may not index if**:
- Thin content (< 300 words)
- Duplicate content
- Doorway pages
- Low-quality content

**Check**:
- Ensure each page has unique, valuable content
- Minimum 500-1000 words for main pages
- Clear topic focus and keyword targeting

#### C. Mobile Usability
**Issues**:
- Not mobile-friendly = indexing delays
- Check: Google Mobile-Friendly Test
- Ensure responsive design
- No interstitials blocking content

## Systematic Troubleshooting Checklist

### Phase 1: Immediate Checks (1 hour)
- [ ] Check robots.txt for blocks
- [ ] Verify no noindex tags in <head>
- [ ] Confirm X-Robots-Tag headers
- [ ] Test homepage in URL Inspection Tool
- [ ] Verify sitemap accessibility
- [ ] Check WordPress Reading Settings

### Phase 2: Technical Audit (2-4 hours)
- [ ] Crawl site with Screaming Frog
- [ ] Validate all canonicals
- [ ] Check all HTTP status codes
- [ ] Validate sitemap XML format
- [ ] Test page speed (PageSpeed Insights)
- [ ] Check JavaScript rendering
- [ ] Verify mobile responsiveness

### Phase 3: Search Console Deep Dive (1-2 hours)
- [ ] Review Coverage report errors
- [ ] Check for manual actions
- [ ] Submit/resubmit sitemap
- [ ] Request indexing for key pages
- [ ] Check Core Web Vitals
- [ ] Review crawl stats

### Phase 4: Advanced Diagnosis (2-4 hours)
- [ ] Check server logs for Googlebot access
- [ ] Verify DNS configuration
- [ ] Test international targeting settings
- [ ] Review hreflang implementation (if multilingual)
- [ ] Check for duplicate content issues
- [ ] Analyze content quality

### Phase 5: Ongoing Monitoring (Daily/Weekly)
- [ ] Monitor `site:` search results
- [ ] Check Search Console daily
- [ ] Track ranking for brand name
- [ ] Monitor crawl rate in Search Console
- [ ] Review index coverage trends

## Common Solutions Summary

| Issue | Solution | Time to Fix |
|-------|----------|-------------|
| Robots.txt blocking | Update robots.txt | Instant |
| Noindex tags | Remove noindex | 1-2 weeks |
| Sitemap errors | Fix XML format | 1-3 days |
| Slow server | Upgrade hosting / optimize | 1 day |
| No canonical tags | Add canonicals | 1-2 weeks |
| Poor content | Improve quality | 2-4 weeks |
| New domain | Build authority + wait | 3-6 months |
| JavaScript issues | Implement SSR | 1-2 weeks |

## Conclusion

Indexing issues are almost always caused by one or more of these factors. Work through this checklist systematically, starting with the easiest checks first. Most issues can be resolved within 1-2 weeks, but new domains naturally take longer to gain trust and indexing priority from Google.

