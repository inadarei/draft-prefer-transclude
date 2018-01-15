# Draft: HTTP Prefer Transclude

Draft proposal to standardize "transclude" preference for the [RFC7240 Prefer](https://tools.ietf.org/search/rfc7240) header

Link to the HTML version of the latest spec: http://spec.transclude.io

## Workspace Setup 

```
> git clone https://github.com/inadarei/draft-prefer-transclude.git
> gem install kramdown-rfc2629
> sudo easy_install pip # optional, if you don't already have it
> sudo pip install xml2rfc
> .githooks/install.sh # To enable automatic rebuilds on commit
```

## Using

1. Edit draft.md
2. To regenerate the latest version of XML/TXT/HTML;

    ```
    make latest
    ```