Odoo is usually extended internally via modules, but many of its features and all of its data are also available from the outside for external analysis or integration with various tools.

[![PyPI version](https://badge.fury.io/py/odooapiclient.svg)](https://badge.fury.io/py/odooapiclient)

🐍 Python client library for Odoo External API

## Installation

Install this library in a [virtualenv](https://virtualenv.pypa.io/en/latest/) using pip.

### Mac/Linux

```
pip install odooapiclient
```

### Windows

```
<your-env>\Scripts\pip.exe install odooapiclient
```

## Supported Python Versions

Python 3.4, 3.5, 3.6 and 3.7

## Authenticate

```
from odooapiclient.client import Client
client = Client(host='andriisem.odoo.com', dbname='andriisem', ssl=True)
client.authenticate(login='', pwd='')
```

## Search

```
client.search('res.country', [('name', '=', country_name)])
```

## Create

```
res_partner_id = client.create('account.payment.term', {
    'name': payment_terms,
    'line_ids': [(0, 0, {
        'value': 'balance',
        'days': days
    })]
})
```

*More features coming soon...*