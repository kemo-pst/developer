# unittest discovery in fresh repos

When validating a Python repo that may not have pytest installed, prefer explicit unittest discovery over a plain `python -m unittest -v`.

## Why

A plain `python -m unittest -v` can return:

- `Ran 0 tests`
- `NO TESTS RAN`

...even when test files exist in `tests/`.

## Use this instead

```bash
python -m unittest discover -s tests -p 'test*.py' -v
```

That produces a real discovery pass and a trustworthy exit code.

## Observed workflow

- project had `tests/test_demo.py`
- `python -m unittest -v` found no tests
- `python -m unittest discover -s tests -p 'test*.py' -v` correctly ran all tests

## Rule of thumb

- Use pytest when the repo already depends on it.
- Otherwise, use explicit unittest discovery for verification in a clean environment.
