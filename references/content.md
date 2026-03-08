# Content / Marketing Site Architecture

For blogs, CMS frontends, landing pages, and SEO-focused React apps.

> **Framework note:** If using Next.js, Remix, or Astro, follow the framework's routing and meta conventions first. This reference is for React SPA (Vite + React Router) projects. The splitting principles still apply regardless of framework.

## Directory Structure

```
client/src/
  App.tsx                          ← Routes (≤80 lines)
  main.tsx
  layouts/
    SiteLayout.tsx                 ← Header + Footer + Outlet
    ArticleLayout.tsx              ← Article-specific layout (sidebar, TOC)
  pages/
    home/
      index.tsx                    ← Landing page
      HeroSection.tsx
      FeatureCards.tsx
      Testimonials.tsx
    blog/
      index.tsx                    ← Article list with pagination
      [slug].tsx                   ← Article detail (or dynamic route)
      ArticleCard.tsx              ← List item component
      useArticles.ts               ← Fetch + filter hook
    about/
      index.tsx
    contact/
      index.tsx
      ContactForm.tsx
      useContact.ts
  components/
    SEOHead.tsx                    ← Meta tags, OG, JSON-LD
    LazyImage.tsx                  ← Intersection Observer lazy loading
    ShareButtons.tsx               ← Social sharing
    Newsletter.tsx                 ← Email signup
    Breadcrumb.tsx
    Pagination.tsx
  hooks/
    useSEO.ts                      ← Dynamic meta tag management
    useIntersection.ts             ← Lazy loading / animation triggers
  services/
    api.ts
    contentApi.ts                  ← CMS/content endpoints
    analyticsApi.ts                ← Page view tracking
  styles/
    global.css
    typography.css                 ← Article typography
    variables.css
```

## Content-Specific Patterns

### SEO Head Component (SPA)

For SPAs without a framework, use `react-helmet-async` or similar. If using Next.js/Remix, use the framework's built-in `<head>` management instead.

```tsx
// components/SEOHead.tsx — SPA example
// Replace with your framework's meta approach if applicable

export default function SEOHead({
  title, description, image, url, type = 'website', article,
}: {
  title: string;
  description: string;
  image?: string;
  url?: string;
  type?: string;
  article?: { publishedAt: string; author: string };
}) {
  const siteName = 'Your Site';
  const fullTitle = title ? `${title} | ${siteName}` : siteName;

  // Update document head directly (or use react-helmet-async)
  useEffect(() => {
    document.title = fullTitle;
    setMeta('description', description);
    setMeta('og:title', fullTitle);
    setMeta('og:description', description);
    if (image) setMeta('og:image', image);
    if (url) setMeta('og:url', url);
    setMeta('og:type', type);

    if (article) {
      setJsonLd({
        '@context': 'https://schema.org',
        '@type': 'Article',
        headline: title,
        description,
        image,
        datePublished: article.publishedAt,
        author: { '@type': 'Person', name: article.author },
      });
    }
  }, [title, description, image, url, type, article]);

  return null;
}

function setMeta(name: string, content: string) {
  const attr = name.startsWith('og:') ? 'property' : 'name';
  let el = document.querySelector(`meta[${attr}="${name}"]`);
  if (!el) {
    el = document.createElement('meta');
    el.setAttribute(attr, name);
    document.head.appendChild(el);
  }
  el.setAttribute('content', content);
}

function setJsonLd(data: object) {
  let el = document.querySelector('script[data-seo-jsonld]');
  if (!el) {
    el = document.createElement('script');
    el.setAttribute('type', 'application/ld+json');
    el.setAttribute('data-seo-jsonld', '');
    document.head.appendChild(el);
  }
  el.textContent = JSON.stringify(data);
}
```

### Article Page Split

```tsx
// pages/blog/[slug].tsx — ≤400 lines
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
