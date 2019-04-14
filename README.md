# Website contents

## Usage

```bash
# 1. Fetching the necessary repositories
# website: static content (articles, pics)
# teuze.github.io: Hugo output (html shit)
# hello-friend-ng: badass theme by rhazdon

git clone https://github.com/teuze/website
cd website
git clone https://github.com/rhazdon/hugo-theme-hello-friend-ng themes/hello-friend-ng
git clone https://github.com/teuze/teuze.github.io public
```
```bash
# 2. Managing your blog locally

hugo		# builds on /public
hugo serve	# same + hosts on localhost:1313

hugo new posts/yourpost.md
```
```bash
# 3. Saving changes and pushing to remote

git add posts/
git commit -m "New Article on Baby Penguins"
git push

cd public/
git add .
git commit -m "New Article on Baby Penguins"
git push
```
