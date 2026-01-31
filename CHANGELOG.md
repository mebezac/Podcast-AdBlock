# 1.0.0 (2026-01-31)


### Bug Fixes

* add type parameters for dict return types in posts.py ([dc3f05f](https://github.com/mebezac/Podcast-AdBlock/commit/dc3f05f1607f4e510f7006aa9d8aaaf53c093330))
* disable too-many-lines pylint warning for post_routes ([8b6e8ff](https://github.com/mebezac/Podcast-AdBlock/commit/8b6e8fffeb85bebe1606ad378b8d7a63c7af8faf))
* feed urls were not being properly generated with reverse proxy settings ([be7dbfd](https://github.com/mebezac/Podcast-AdBlock/commit/be7dbfdf65157cc601028e1eb373a541266504dd))
* reformat completed by ci.sh ([9dae208](https://github.com/mebezac/Podcast-AdBlock/commit/9dae20816d7485048ac13f74b6671019eb9b88ba))
* reformat docstrings to match black style ([13fe72d](https://github.com/mebezac/Podcast-AdBlock/commit/13fe72d7e105413bc9581864880d2fda81784cee))
* remove duplicate import statements in writer actions __init__.py ([6349fae](https://github.com/mebezac/Podcast-AdBlock/commit/6349fae6dcb663f90468c05c7cafe74c0253dc10))
* resolve mypy type errors and disable docker push for PRs ([f6fafcb](https://github.com/mebezac/Podcast-AdBlock/commit/f6fafcbe1b250cf6d8f0bc6a42cbaed9de4ae93e))
* resolve pylint warnings ([3ba61eb](https://github.com/mebezac/Podcast-AdBlock/commit/3ba61ebf1cef4eafce0279d87a0b49875490205e))
* sort imports in writer actions __init__.py for isort ([1867d75](https://github.com/mebezac/Podcast-AdBlock/commit/1867d755fc2ceb15835437606c8eb697e0303f94))
* update container name and network name to podly-pure-podcasts across all compose files ([4a4603e](https://github.com/mebezac/Podcast-AdBlock/commit/4a4603efa07ba95b6d0123cfa72ff68447df8310))
* update test to reflect new API key behavior ([8b8456f](https://github.com/mebezac/Podcast-AdBlock/commit/8b8456f9f970e2b4750410781cd8058c9a9a7630))


### Features

* add retryable processing steps and separate API keys ([057e584](https://github.com/mebezac/Podcast-AdBlock/commit/057e58476812ee0b5081098998550e5cdaa091cd)), closes [#1](https://github.com/mebezac/Podcast-AdBlock/issues/1) [#2](https://github.com/mebezac/Podcast-AdBlock/issues/2)
* add support for local LLMs ([c2ded6b](https://github.com/mebezac/Podcast-AdBlock/commit/c2ded6b9eff1d7f4d8665be543255d74b8baf13f))
* add support for local LLMs ([0320f24](https://github.com/mebezac/Podcast-AdBlock/commit/0320f24c9b121c04f264b57d8456c7cecaf43776))
* **processing:** add JobManager; refactor processor/API/UI; remove legacy jobs ([01a139f](https://github.com/mebezac/Podcast-AdBlock/commit/01a139f340ea7cf6221341598d459ee1c6c396c0))


### Reverts

* test_process_audio.py ([bbbd0f1](https://github.com/mebezac/Podcast-AdBlock/commit/bbbd0f18e85710820947ba3f4fd35d9a40b390ae))
