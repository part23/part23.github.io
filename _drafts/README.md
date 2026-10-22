# 새 글 작성 가이드

`_posts/YYYY-MM-DD-제목.md` 로 파일을 만들고 아래 front matter를 붙입니다.

```yaml
---
title: 글 제목
date: 2026-10-22 12:00:00 +0900
categories: [Exploit, Google]   # [대분류, 소분류] — 소분류: Google / Apple / Firefox / Samsung / Trend
tags: [chrome, v8, cve-2026-xxxx]
---
```

## 한/영 버전

- 한국어: `_posts/YYYY-MM-DD-slug.md` — 태그에 `korean` 추가, 상단에 영문 링크
- 영어: `_posts/YYYY-MM-DD-slug-en.md` — front matter에 아래 추가

```yaml
lang: en
permalink: /en/posts/slug/
tags: [english, ...]
```

양쪽 글 맨 위에 서로 링크:

```markdown
> 🇺🇸 [Read in English](/en/posts/slug/)
{: .prompt-info }
```
