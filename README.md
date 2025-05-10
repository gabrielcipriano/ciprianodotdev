## installation

follow the hugo installation guide: https://gohugo.io/installation/

## draft lifecycle

1. `checkout drafts`
2. create a new post with `hugo new posts/<post-name>.md`
3. commit and push to the `drafts` branch
4. Go to [cloud flare pages](https://pages.cloudflare.com/) and see the preview
5. If happy, cherry-pick the commit to the `main` branch.
6. change to `draft = false` in the post front matter

obs. DO NOT MERGE THE DRAFTS BRANCH INTO MAIN, as it has a different `build.sh` file that publishes the draft posts.

## development mode
    
```bash
hugo server -b http://localhost -p 8000    
```

## build

```bash
hugo
```

## to publish

Just push to the main branch and the [cloufare pages](https://pages.cloudflare.com/) will update automatically

Cloud flare pages builds based on the file `build.sh` in the root directory. 
