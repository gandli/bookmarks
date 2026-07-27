# 📑 bookmarks

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="assets/readme/hero.svg">
  <img alt="gandli/bookmarks — version-controlled browser bookmarks backup" src="assets/readme/hero.svg" width="100%">
</picture>

Version-controlled backup of Chrome bookmarks from the **gandli** browser profile.  
Includes **5,080 GitHub starred repositories** imported from the GitHub API.

## Structure

| Category | Count | Content |
|----------|-------|---------|
| 📁 日常 | 73 | 社交·邮箱·购物·地图·影音·音乐·云盘·内网VPN |
| 📁 AI | 118 | AI对话·编程·绘画·API |
| 📁 CTF | 103 | 平台·编码·渗透·工具·知识 |
| 📁 开发 | 5,191 | GitHub x 5,080·前端·后端·移动端·工具链·其他 |
| 📁 设计 | 170 | 图标·配色·字体·插画·原型·灵感·样机·其他 |
| 📁 云服务 | 37 | VPS·国内云·BaaS·其他 |
| 📁 知识库 | 115 | Notion·笔记·博客周刊·阅读·教程 |
| 📁 工具 | 149 | 开发·在线·图片·网络·代理·政府·系统·临时邮箱 |
| **Total** | **5,956** | **57 folders** |

## Files

| File | Description |
|------|-------------|
| `AccountBookmarks` | Chrome native JSON — drop-in replacement for `~/Library/.../AccountBookmarks` |
| `bookmarks_flattened.csv` | Flat table: `folder\|name\|url` |
| `SUMMARY.md` | Human-readable tree structure |

## Usage

### Backup

```bash
cp "$HOME/Library/Application Support/Google/Chrome/Default/AccountBookmarks" \
  backups/bookmarks_$(date +%Y%m%d_%H%M%S).json
```

### Restore

```bash
cp AccountBookmarks ~/Library/Application\ Support/Google/Chrome/Default/
```

### Export

```bash
# CSV
jq -r '.roots.bookmark_bar.children[] | select(.type == "folder")
  | .name as $f | .children[]? | select(.type == "url")
  | [$f, .name, .url] | @csv' AccountBookmarks > bookmarks.csv

# Tree
jq -r '.roots.bookmark_bar' AccountBookmarks | python3 -c '
import sys, json
d = json.load(sys.stdin)
def w(n,i=0):
  for c in n.get("children",[]):
    if c["type"]=="folder":
      print("  "*i + "📁 " + c["name"] + f" ({len(c.get(\"children\",[]))})")
      w(c,i+1)
w(d)
'
```

### Sync GitHub starred repos

```bash
gh api users/gandli/starred --paginate --jq '.[] | "\(.html_url) \(.full_name)"' | \
while read url name; do
  python3 -c "
import json, hashlib
with open('AccountBookmarks') as f: d = json.load(f)
gh = [c for c in d['roots']['bookmark_bar']['children']
      if c['name']=='GitHub' and c['type']=='folder'][0]
exist = {c['url'] for c in gh['children'] if c.get('url')}
if '$url' not in exist:
  gh['children'].append({'name':'$name','type':'url','url':'$url'})
  d['checksum'] = hashlib.md5(json.dumps(d,indent=3).encode()).hexdigest()
  json.dump(d, open('AccountBookmarks','w'), indent=3, ensure_ascii=False)
"
done
```

## License

The bookmark data is user-collected content. The backup scripts and tooling are MIT.
