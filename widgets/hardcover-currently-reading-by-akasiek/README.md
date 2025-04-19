## Preview

![](preview.png)

## Configuration

```yaml
- type: custom-api
  title: Hardcover
  cache: 3h
  url: https://api.hardcover.app/v1/graphql
  headers:
    content-type: application/json
    authorization: ${HARDCOVER_API_KEY}
  body:
    query: |
      query MyQuery {
        me {
          user_books(
            where: {user_book_reads: {id: {_is_null: false}, finished_at: {_is_null: true}, paused_at: {_is_null: true}, started_at: {_is_null: false}}}
          ) {
            id
            user_book_reads(
              where: {finished_at: {_is_null: true}, paused_at: {_is_null: true}, started_at: {_is_null: false}}
            ) {
              id
              started_at
              progress
              edition {
                image {
                  url
                }
                contributions(where: {contribution: {_is_null: true}}) {
                  contribution
                  author {
                    name
                  }
                }
              }
            }
            book {
              title
              slug
            }
          }
        }
      }
  template: |
    <ul class="list list-gap-10 collapsible-container" data-collapse-after="5">
      {{ range .JSON.Array "data.me.0.user_books" }}
        <li class="flex items-center gap-10">
          <div style="border-radius: 5px; max-height: 10rem; max-width: 6rem; overflow: hidden;">
            <img src="{{ .String "user_book_reads.0.edition.image.url" }}" class="card" style="width: 100%; height: 100%; object-fit: cover; object-position: center;">
          </div>
          <div class="flex-1">
            <a class="size-h4 color-highlight block text-truncate" href="https://hardcover.app/books/{{ .String "book.slug" }}">{{ .String "book.title" }}</a>
            <ul class="list-horizontal-text size-h5">
              {{ range .Array "user_book_reads.0.edition.contributions" }}
                <li>{{ .String "author.name" }}</li>
              {{ end }}
            </ul>
            <ul class="list-horizontal-text" >
              <li>{{ .String "user_book_reads.0.started_at" }}</li>
              <li>{{ .Int "user_book_reads.0.progress" }}%</li>
            </ul>
          </div>
        </li>
      {{ end }}
    </ul>
```

## Environment Variables

- `HARDCOVER_API_KEY` - Hardcover API authorization token, which you can obtain from [here](https://hardcover.app/account/api). Information will be fetched for the user associated with the token.
