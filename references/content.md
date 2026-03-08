# Content / Marketing Site Architecture

For blogs, CMS frontends, landing pages, and SEO-focused React apps.

## Directory Structure

```
client/src/
  App.jsx                          ← Routes (≤80 lines)
  main.jsx
  layouts/
    SiteLayout.jsx                 ← Header + Footer + Outlet
    ArticleLayout.jsx              ← Article-specific layout (sidebar, TOC)
  pages/
    home/
      index.jsx                    ← Landing page
      HeroSection.jsx
      FeatureCards.jsx
      Testimonials.jsx
    blog/
      index.jsx                    ← Article list with pagination
      [slug].jsx                   ← Article detail (or dynamic route)
      ArticleCard.jsx              ← List item component
      useArticles.js               ← Fetch + filter hook
    about/
      index.jsx
    contact/
      index.jsx
      ContactForm.jsx
      useContact.js
  components/
    SEOHead.jsx                    ← Meta tags, OG, JSON-LD
    LazyImage.jsx                  ← Intersection Observer lazy loading
    ShareButtons.jsx               ← Social sharing
    Newsletter.jsx                 ← Email signup
    Breadcrumb.jsx
    Pagination.jsx
  hooks/
    useSEO.js                      ← Dynamic meta tag management
    useIntersection.js             ← Lazy loading / animation triggers
  services/
    api.js
    contentApi.js                  ← CMS/content endpoints
    analyticsApi.js                ← Page view tracking
  styles/
    global.css
    typography.css                 ← Article typography
    variables.css
```

## Content-Specific Patterns

### SEO Head Component

```jsx
// components/SEOHead.jsx
import { Helmet } from 'react-helmet-async';

export default function SEOHead({ title, description, image, url, type = 'website', article }) {
  const siteName = 'Your Site';
  const fullTitle = title ? `${title} | ${siteName}` : siteName;

  return (
    <Helmet>
      <title>{fullTitle}</title>
      <meta name="description" content={description} />
      <meta property="og:title" content={fullTitle} />
      <meta property="og:description" content={description} />
      <meta property="og:image" content={image} />
      <meta property="og:url" content={url} />
      <meta property="og:type" content={type} />
      {article && (
        <script type="application/ld+json">
          {JSON.stringify({
            '@context': 'https://schema.org',
            '@type': 'Article',
            headline: title,
            description,
            image,
            datePublished: article.publishedAt,
            author: { '@type': 'Person', name: article.author },
          })}
        </script>
      )}
    </Helmet>
  );
}
```

### Article Page Split

```jsx
// pages/blog/[slug].jsx — ≤400 lines
import { useParams } from 'react-router-dom';
import { useArticle } from './useArticle';
import SEOHead from '../../components/SEOHead';
import ShareButtons from '../../components/ShareButtons';
import ArticleContent from './ArticleContent';
import RelatedArticles from './RelatedArticles';

export default function ArticlePage() {
  const { slug } = useParams();
  const { article, related, loading } = useArticle(slug);

  if (loading) return <div>Loading...</div>;
  if (!article) return <div>Not found</div>;

  return (
    <>
      <SEOHead
        title={article.title}
        description={article.excerpt}
        image={article.coverImage}
        type="article"
        article={article}
      />
      <ArticleContent article={article} />
      <ShareButtons url={window.location.href} title={article.title} />
      <RelatedArticles articles={related} />
    </>
  );
}
```

## Performance Checklist for Content Sites

- [ ] All images use `<LazyImage>` with Intersection Observer
- [ ] Routes are lazy-loaded (`React.lazy`)
- [ ] CSS is split per page (CSS modules or dynamic imports)
- [ ] JSON-LD structured data on article pages
- [ ] `<link rel="canonical">` on every page
- [ ] Sitemap generation (build-time or API)
