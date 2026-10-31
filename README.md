# walkasjesus_translations

This repository contains all translation files from English to other languages. We use Google translate to make a first automated translation. The next step for the translator is to review all translated items and acknowledge them through the admin panel of the website. 

_**All Bible references are automatically imported with the configured Bible Translation for that language**_

## Translators translation manual

If you think this website contains valuable information to help in Discipling people to become Followers of Christ, you are very welcome to translate this website into your own language.  

In order to translate the website, you can do the following:

1. Register for an account on the [website](https://www.walkasjesus.org/account/signup)
2. Once your account is acknowledged, you can [login here](https://www.walkasjesus.org/account/login/)
3. Click on the word `Translations` to go to the translation page
4. Click on the link `walkasjesus_website` under the title of the language where you want to translate
5. Start translation for all the fields. `Fuzzy` entries call for revision by the translator. When an entry is revised, you can deselect the fuzzy field. Once you have done all the items, you can click on `save and translate next block`.

## Adding completely new languages

If you want to add completely new languages you need to perform the following steps.  
In this example we will add the _Dutch_ language, which will have **nl** as language code (Format should be in [ISO 639-1](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes)).

1. Add the new language to the `base_settings.py` of the `walkasjesus_framework`

```
LANGUAGES = [
    ('en', gettext_lazy('English')),  # First language is the default for modeltranslation
    ('nl', gettext_lazy('Dutch')),
]
...


# Rosetta settings
ROSETTA_SHOW_AT_ADMIN_PANEL = True
ROSETTA_LANGUAGE_GROUPS = [
    'translators-nl',
]
```

2. Activate your virtual environment:  
`source ./venv/bin/activate`
3. Refresh the catalog and preview the interface workload without calling Google:
`./update_translation_files.sh nl --scope ui --limit 50 --dry-run`
This extracts template/Python messages and synchronizes database-backed fields.
4. Compile the .po files. Run this also after changing the .po files.  
`django-admin compilemessages`  
_Or alternatively with msgfmt:_  
`msgfmt translations/locale/nl/LC_MESSAGES/django.po -o translations/locale/nl/LC_MESSAGES/django.mo`
5. Translate the interface first, in a small batch:
`./auto_translate_files.sh nl --scope ui --limit 50 --request-interval 2`
Then translate authorized study content separately:
`./auto_translate_files.sh nl --scope content --limit 25 --request-interval 2`

Both scripts preserve their existing default of all scopes/locales when no
options are provided. Explicit locale, scope and limit options are recommended
for new languages.

6. The translator protects Django placeholders and HTML tags. If Google changes
a protected token, the run stops with an error rather than storing damaged text.
Completed entries are saved after each entry and compiled even when a translation
run fails. Correct damaged translations manually before resuming.

7. Now the translator can start translating the files. Follow the steps in paragraph [Translators translation manual]

## Interface versus study content

Keep a single `django.po`/`django.mo` catalog per language so Django's gettext,
Vinaigrette and Rosetta resolve messages exactly as before. The separation is in
the translation workload, not in runtime catalogs:

- `--scope ui`: messages referenced by templates or Python, including settings,
  navigation, page instructions and model-choice labels.
- `--scope content`: database-backed messages such as step titles, lesson stories,
  activities, questions and concise Law of Messiah labels. Synchronization adds
  `model:Model.field` source references; legacy messages without references also
  belong to this scope. A message shared with the interface is handled by `ui`.
- `--scope all`: both workloads (the backwards-compatible default).

Long Law of Messiah commentary bodies are intentionally not registered as PO
translations. Bible verse text comes from the selected Bible translation, not
from Google. Only send content to a translation service when you have permission
to do so.

`--limit` caps entries across the entire run, not per file or language. An entry
can require several requests when it contains paragraphs or plural forms.
`--request-interval` sets minimum seconds between request starts, including
requests within a long entry. The default is 1 second. Use `--dry-run` to see the
remaining UI/content counts without sending requests or changing translations.
The refresh script still refreshes the catalog in dry-run mode.

### Continuing Hebrew

From the website directory:

```bash
./update_translation_files.sh he --scope ui --limit 25 --dry-run
./auto_translate_files.sh he --scope ui --limit 25 --request-interval 2
./auto_translate_files.sh he --scope content --limit 25 --request-interval 2
```

Repeated successful batches skip completed translations. Fuzzy entries remain
eligible for automatic translation; reviewed entries are preserved.

### Local/offline Hebrew drafts

The `local` provider runs the Argos English-to-Hebrew 1.5 model directly through
CTranslate2 and SentencePiece on the CPU. It does not call a translation API or
download anything at inference time. Dependencies and the model are installed
once; the website server does not need the inference dependencies.

Install the optional dependencies in a translation virtual environment:

```bash
python3 -m venv --system-site-packages ~/.cache/walkasjesus/translation-venv
~/.cache/walkasjesus/translation-venv/bin/python -m pip install -r ../translate_tool/requirements-local.txt
```

The environment must also contain the site's existing Django, polib and
translate_tool dependencies. `--system-site-packages` reuses them when they are
already installed in the selected Python interpreter.

Download `translate-en_he-1_5.argosmodel` from the
[official Argos package index](https://github.com/argosopentech/argospm-index).
It is a ZIP archive of approximately 120 MiB. Extract it outside the repository,
for example into `~/.cache/walkasjesus/translation-models/`. Retain its README
and attribution information. The model directory must contain `metadata.json`,
`sentencepiece.model` and `model/model.bin`. This is a model trained on public
parallel corpora, not a specialist theological translator; review terminology
and meaning carefully.

From the website directory:

```bash
export WAJ_TRANSLATION_PYTHON="$HOME/.cache/walkasjesus/translation-venv/bin/python"
./auto_translate_files.sh he --provider local \
  --model-path "$HOME/.cache/walkasjesus/translation-models/translate-en_he-1_5" \
  --scope ui --limit 50
./auto_translate_files.sh he --provider local \
  --model-path "$HOME/.cache/walkasjesus/translation-models/translate-en_he-1_5" \
  --scope content --limit 50
```

Both scripts accept `--provider local` and `--model-path`. Omit `--limit` to
translate all pending entries in the selected scope. `--dry-run` reports pending
counts and drafts awaiting review without loading the model. Protected
placeholders/HTML are kept out of inference and retained exactly. Splitting
around those tokens can reduce sentence context, so inspect such entries in
particular. Missing models, unsupported language pairs and inference errors
stop the run explicitly; there is no fallback to Google.

#### Reviewing and publishing in Rosetta

1. Sign in as a superuser or an authorized Hebrew translator
   (`translators-he` under the existing Rosetta permission settings).
2. Open `/rosetta/` and choose Hebrew and the main project catalog. For the
   account catalog, use `/rosetta/files/all/`: Rosetta classifies the sibling
   account app outside the project filter. Review both catalogs.
3. Select the **Fuzzy** filter. Local drafts carry the translator comment
   `Machine translation draft: local Argos; review in Rosetta.`
4. Correct the Hebrew and clear **Fuzzy** only after approval. Save the page.
   Rosetta saves the PO and recompiles the MO.
5. Reload/restart deployed Django workers after publishing catalogs, since
   loaded gettext catalogs may remain cached.

Fuzzy drafts are deliberately excluded from compiled MO files, so the site
continues to show the source English until approval. Restarting automatic
translation skips completed machine drafts as well as approved translations,
including when using the Google provider. To regenerate a rejected draft,
clear its translation text first. Plural forms are draft approximations and
must be checked individually in Rosetta.

### Google rate limits

The current adapter uses Google's unauthenticated web translation endpoint,
not an authenticated Cloud Translation API project with a purchased quota.
Slower, smaller runs reduce traffic but cannot guarantee that Google accepts it.
On HTTP 429 the command stops, preserves completed work, reports `Retry-After`
when available, and exits unsuccessfully. Do not repeatedly restart or change
networks to evade the limit. Wait for the restriction to clear. For sustained
bulk translation, consider an authenticated translation API with explicit quota
and billing rather than this endpoint.

## Related projects and repositories

The following projects are related to this repository.

### walkasjesus_framework

This repository is the heart of the application which will show all the different components of the Walk as Jesus Application in a fancy Python Framework [Django](https://www.djangoproject.com/)

### walkasjesus_server

This repository contains IT Automation tools for [Ansible](https://docs.ansible.com/ansible/latest/index.html) to install & configure all the server components which are required to serve the [Walk as Jesus Framework](https://github.com/walkasjesus/walkasjesus_framework)

### walkasjesus_biblereferences

Repository for the Walk as Jesus Framework where all the commandments with all their related Bible References are stored in a CSV. This CSV can be imported/exported with the [Walk as Jesus Framework](https://github.com/walkasjesus/walkasjesus_framework) and the [Walk as Jesus Server](https://github.com/walkasjesus/walkasjesus_server)

### walkasjesus_media

Repository for the [Walk as Jesus Framework](https://github.com/walkasjesus/walkasjesus_framework) where all the resources (movies, songs, blogs, sermons, testimonies, etc) in all languages are stored in a CSV. This CSV can be imported/exported with the [Walk as Jesus Framework](https://github.com/walkasjesus/walkasjesus_framework) and the [Walk as Jesus Server](https://github.com/walkasjesus/walkasjesus_server)

### LawofMessiah

This repository contains the structured content and scripts for processing the commandments (*mitzvot*) derived from the Bible, as presented in the book series *The Law of Messiah - Torah from a New Covenant Perspective* by Michael Rudolph with Daniel C. Juster. The goal of this project is to make the commandments more accessible by organizing them into structured formats (e.g., YAML) and providing tools for filtering, searching, and studying them.
[Law of Messiah](https://github.com/walkasjesus/LawofMessiah)