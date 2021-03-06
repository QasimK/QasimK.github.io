# QasimK

My personal site is at <https://QasimK.io>.

## Development

Note: Prefix urls with `{{ '/link' | relative_url }}`.

Once in while we should check that our dependencies match GitHub pages:
* <https://pages.github.com/versions/>
* <https://github.com/github/pages-gem/releases>

### Setup

1. [Install RVM](https://rvm.io/rvm/install) - Qasim, make sure you follow the full instructions this time. (Note: Fish Shell https://rvm.io/integration/fish)
2. `rvm install (cat .ruby-version)`
3. `rvm gemset create (cat .ruby-gemset)`

Note: `cd`ing into this project root will activate the development environment
due to `.ruby-gemset` and `.ruby-version`.

4. `gem install bundler`
5. `bundle` to install dependencies

### Development process

* `rvm gemset use qasimk`.
* `make watch` and browse over to <http://localhost:4000>.

## Deployment

The website will automatically deploy when pushing master to
<https://github.com/QasimK/qasimk.github.io>. The website will be available
at <https://qasimk.io>.

GitHub will automatically build the website, and a configured DNS CNAME record
allows them to host the website, but they do not support HTTPS on custom domains
(yet).

## Tools

* <http://www.favicomatic.com/>

## Structure

Maybe something like

qasimk.io/site-map/
qasimk.io/2018/wii-games-on-pc/
qasimk.io/talks/2018/database-migrations/
qasimk.io/tech-notes/2018/linux-email/
qasimk.io/ideas/a-cycle-friendly-city/
qasimk.io/essays/a-driverless-vision/
qasimk.io/tags/
qasimk.io/tags/linux/
qasimk.io/scribbles/
qasimk.io/cool/
qasimk.io/comments/
qasimk.io/comments/speculative/the-art-of-thinking-clearly/
qasimk.io/comments/fiction/mistborn/
qasimk.io/comments/films/spirited-away/
qasimk.io/comments/games/bastion/
qasimk.io/comments/series/battlestar-galactica/
qasimk.io/comments/anime/psycho-pass/
qasimk.io/comments/tabletop/munchkin/

Blog categories: Tech Notes (+Talks), Essays, Ideas
Different styles.

Blog could possibly be sub-divided into technical articles and not?
Blog has tags.
Scribbles will be moved to twitter. (???)

What's the point of tags?
