# Photon

Jump to:

- [Self-hosting](#self-hosting)
- [Public Instances](#public-instances)

An alternative web client for Lemmy with the UI of Xylo

## Screenshots

![image](https://github.com/Xyphyn/photon/assets/80978739/dab88315-691b-4471-add7-321b8a803c54)

## Self-hosting

Put Photon on your own domain for easier onboarding.

### Running from image

```sh
# Use the env var PUBLIC_INSTANCE_URL to set the default instance URL.
docker run -p 8080:3000 -e PUBLIC_INSTANCE_URL=example.com ghcr.io/xyphyn/photon:latest
```

### Running from repo

**More unstable but latest features**

Clone the repo:

```sh
git clone https://github.com/Xyphyn/photon && cd photon
```

and run the docker container:

```sh
docker build -t photon .

# Use the env var PUBLIC_INSTANCE_URL to set the default instance URL.
docker run -p 8080:3000 -it photon:latest
```

There you go, you now have an instance of Photon running.

### Configuring default settings

The following environment variables can be set to override the default settings:

| Variable                        | Values              | Default Value                          |
| ------------------------------- | ------------------- | -------------------------------------- |
| PUBLIC_INSTANCE_URL             | URL                 | `lemmy.ml`                             |
| PUBLIC_LOCK_TO_INSTANCE         | `bool`              | `true` if `PUBLIC_INSTANCE_URL` is set |
| PUBLIC_SSR_ENABLED              | `bool`              | `false`                                |
| PUBLIC_THEME                    | system\|dark\|light | system                                 |
| PUBLIC_EXPANDABLE_IMAGES        | `bool`              | true                                   |
| PUBLIC_MARK_READ_POSTS          | `bool`              | true                                   |
| PUBLIC_REVERT_VOTE_COLORS       | `bool`              | false                                  |
| PUBLIC_SHOW_INSTANCES_USER      | `bool`              | false                                  |
| PUBLIC_SHOW_INSTANCES_COMMUNITY | `bool`              | true                                   |
| PUBLIC_SHOW_INSTANCES_COMMENTS  | `bool`              | false                                  |
| PUBLIC_SHOW_COMPACT_POSTS       | `bool`              | false                                  |
| PUBLIC_DEFAULT_FEED_SORT        | `SortType`          | Active                                 |
| PUBLIC_DEFAULT_FEED             | `ListingType`       | Local                                  |
| PUBLIC_DEFAULT_COMMENT_SORT     | `CommentSortType`   | Hot                                    |
| PUBLIC_HIDE_DELETED             | `bool`              | true                                   |
| PUBLIC_HIDE_REMOVED             | `bool`              | false                                  |
| PUBLIC_FULL_WIDTH_LAYOUT        | `bool`              | false                                  |
| PUBLIC_EXPAND_SIDEBAR           | `bool`              | true                                   |
| PUBLIC_DISPLAY_NAMES            | `bool`              | true                                   |
| PUBLIC_NSFW_BLUR                | `bool`              | true                                   |
| PUBLIC_NEW_VOTE_BUTTONS         | `bool`              | false                                  |
| PUBLIC_RANDOM_PLACEHOLDERS      | `bool`              | true                                   |
| PUBLIC_REMOVE_CREDIT            | `bool`              | false                                  |

The values for `SortType`, `ListingType`, and `CommentSortType` are defined by the lemmy-js-client library.

#### Listing Type

https://github.com/LemmyNet/lemmy-js-client/blob/main/src/types/ListingType.ts

- All
- Local
- Subscribed
- Moderator View (not implemented in Photon)

#### Sort Type

(case sensitive)

https://github.com/LemmyNet/lemmy-js-client/blob/main/src/types/SortType.ts

- Active
- Hot
- New
- Old
- TopDay
- TopWeek
- TopMonth
- TopAll
- MostComments
- NewComments
- TopHour
- TopSixHour
- TopTwelveHour (Not implemented in Photon)
- TopThreeMonths (Not implemented in Photon)
- TopSixMonths (Not implemented in Photon)
- TopNineMonths (Not implemented in Photon)
- TopYear (Not implemented in Photon)

#### Comment Sort Type

https://github.com/LemmyNet/lemmy-js-client/blob/main/src/types/CommentSortType.ts
values:

- Hot
- Top
- New
- Old (not implemented in Photon)
- Controversial (not implemented in Photon)

## Public Instances

Want your instance added here? Make a GitHub issue or make a PR! (this is for general purpose Photon instances.)

[phtn.app](https://phtn.app) is the official instance and will get updates instantly.

| Instance                                             | Location     | Contact                                            |
| ---------------------------------------------------- | ------------ | -------------------------------------------------- |
| [phtn.app (Official)](https://phtn.app)              | 🇺🇸 US West   | [photon@xylight.dev](photon@xylight.dev)           |
| [ph.openxng.com](https://ph.opnxng.com)              | 🇸🇬 Singapore | [about.openxng.com](https://about.opnxng.com)      |
| [photon.zhenyapav.com](https://photon.zhenyapav.com) | 🇮🇸 Iceland   | [zhenyapav@zhenyapav.com](zhenyapav@zhenyapav.com) |

## Donate

You can donate at [Buy me a Coffee](https://buymeacoffee.com/xylight)

<a href="https://www.buymeacoffee.com/xylight"><img src="https://img.buymeacoffee.com/button-api/?text=Buy me a coffee&emoji=&slug=xylight&button_colour=FFDD00&font_colour=000000&font_family=Poppins&outline_colour=000000&coffee_colour=ffffff" /></a>
