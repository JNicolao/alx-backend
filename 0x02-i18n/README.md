The topics of this chapter are Internationalization and Localization, commonly abbreviated I18n and L10n. To make my application friendly to people who do not speak English, I'm going to implement a translation workflow that, with the help of language translators, will allow me to offer the application to users in a choice of languages.

The GitHub links for this chapter are: Browse, Zip, Diff.

Introduction to Flask-Babel
As you can probably guess, there is a Flask extension that makes working with translations very easy. The extension is called Flask-Babel and is installed with pip:

(venv) $ pip install "flask-babel<3"
This install the latest 2.x release of Flask-Babel. Flask-Babel is initialized like most other Flask extensions:

app/__init__.py: Flask-Babel instance.

# ...
from flask_babel import Babel

app = Flask(__name__)
# ...
babel = Babel(app)
As part of this chapter, I'm going to show you how to translate the application into Spanish, as I happen to speak that language. I could also work with translators that know other languages and support those as well. To keep track of the list of supported languages, I'm going to add a configuration variable:

config.py: Supported languages list.

class Config(object):
    # ...
    LANGUAGES = ['en', 'es']
I'm using two-letter language codes for this application, but if you need to be more specific, a country code can be added as well. For example, you could use en-US, en-GB and en-CA to support American, British and Canadian English as different languages.

The Babel instance provides a localeselector decorator. The decorated function is invoked for each request to select a language translation to use for that request:

app/__init__.py: Select best language.

from flask import request

# ...

@babel.localeselector
def get_locale():
    return request.accept_languages.best_match(app.config['LANGUAGES'])
Here I'm using an attribute of Flask's request object called accept_languages. This object provides a high-level interface to work with the Accept-Language header that clients send with a request. This header specifies the client language and locale preferences as a weighted list. The contents of this header can be configured in the browser's preferences page, with the default being usually imported from the language settings in the computer's operating system. Most people don't even know such a setting exists, but this is useful as users can provide a list of preferred languages, each with a weight. In case you are curious, here is an example of a complex Accept-Languages header:

Accept-Language: da, en-gb;q=0.8, en;q=0.7
This says that Danish (da) is the preferred language (with default weight = 1.0), followed by British English (en-GB) with a 0.8 weight, and as a last option generic English (en) with a 0.7 weight.

To select the best language, you need to compare the list of languages requested by the client against the languages the application supports, and using the client provided weights, find the best language. The logic to do this is somewhat complicated, but it is all encapsulated in the best_match() method, which takes the list of languages offered by the application as an argument and returns the best choice.

