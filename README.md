# jesus_commandments_translations

This repository contains all translation files from English to other languages. We use Google translate to make a first automated translation. The next step for the translator is to review all translated items and acknowledge them through the admin panel of the website. 

_**All Bible references are automatically imported with the configured Bible Translation for that language**_

## Translators translation manual

If you think this website contains valuable information to help in Discipling people to become Followers of Christ, you are very welcome to translate this website into your own language.  

In order to translate the website, you can do the following:

1. Register for an account on the [website](https://www.jesuscommandments.org/account/signup)
2. Once your account is acknowledged, you can [login here](https://www.jesuscommandments.org/account/login/)
3. Click on the word `Translations` to go to the translation page
4. Click on the link `jesus_commandments_website` under the title of the language where you want to translate
5. Start translation for all the fields. `Fuzzy` entries call for revision by the translator. When an entry is revised, you can deselect the fuzzy field. Once you have done all the items, you can click on `save and translate next block`.

## Adding completely new languages

If you want to add completely new languages you need to perform the following steps.  
In this example we will add the _Dutch_ language, which will have **nl** as language code (Format should be in [ISO 639-1](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes)).

1. Add the new language to the `base_settings.py` of the `jesus_commandments_framework`

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
3. Adding new texts to the .po files:  
`python manage.py makemessages -l nl`
4. Compile the .po files. Run this also after changing the .po files.  
`django-admin compilemessages`
5. Auto translate all the files with Google translate servers by running the following command:  
`auto_translate_files.sh`

_NOTE: If you encounter the following error, this means that Google limited the number of translations you did for this day. You can resolve this to wait for tomorrow and try it again (it will continue where it was), or you can enable a VPN connection and try it again._

```
Expecting value: line 1 column 1 (char 0)
Could not translate all items, stopped on above error.
```

*Make sure to repeat this step until you don't get any errors*

6. When running auto_translate_files.sh Google will translate all django variables in a wrong format. To fix the most errors, run:  
`auto_correct_django_locale_vars.sh`

7. Now the translator can start translating the files. Follow the steps in paragraph [Translators translation manual]

## Related projects and repositories

The following projects are related to this repository.

### jesus_commandments_framework

This repository is the heart of the application which will show all the different components of the Jesus Commandments Application in a fancy Python Framework [Django](https://www.djangoproject.com/)

### jesus_commandments_server

This repository contains IT Automation tools for [Ansible](https://docs.ansible.com/ansible/latest/index.html) to install & configure all the server components which are required to serve the [Jesus Commandments Framework](https://github.com/jesuscommandments/jesus_commandments_framework)

### jesus_commandments_biblereferences

Repository for the Jesus Commandments Framework where all the commandments with all their related Bible References are stored in a CSV. This CSV can be imported/exported with the [Jesus Commandments Framework](https://github.com/jesuscommandments/jesus_commandments_framework) and the [Jesus Commandments Server](https://github.com/jesuscommandments/jesus_commandments_server)

### jesus_commandments_media

Repository for the [Jesus Commandments Framework](https://github.com/jesuscommandments/jesus_commandments_framework) where all the resources (movies, songs, blogs, sermons, testimonies, etc) in all languages are stored in a CSV. This CSV can be imported/exported with the [Jesus Commandments Framework](https://github.com/jesuscommandments/jesus_commandments_framework) and the [Jesus Commandments Server](https://github.com/jesuscommandments/jesus_commandments_server)
