# staal-email

Email through Resend: a [Staal](https://github.com/laurells/staal) package that gives
programs the `email.resend.send` capability, built on Staal's general `net.http`
capability. It talks to [Resend](https://resend.com).

## Authority

This is everything the package can reach. `staalc fetch` prints it when the
package is added, and the compiler refuses the package if its code reaches
beyond it.

| Capability | Targets |
| --- | --- |
| `net.http.post` | `https://api.resend.com/emails` |

## Install

Add the package to your project's `staal.json`, pinned to an exact commit:

```json
{
  "dependencies": {
    "email": {
      "git": "https://github.com/laurells/staal-email",
      "commit": "<full 40-character commit id>"
    }
  }
}
```

Then fetch it. This writes `staal.lock` with the SHA-256 of every file, and every
later check verifies the stored copy against it:

```
staalc fetch
```

## Use

The capability's action:

```
email.resend: send(to: Text @target, subject: Text, text: Text) -> Text
```

A program imports the module and asks for the capability with its own limits:

```
import email.resend

fn notify(to: Text, mail: cap email.resend.send(only: ["*@example.org"], uses: 1)) -> Receipt {
    return mail.send(to, "Your results", "Your results are ready.")
}
```

The operator supplies what the library needs in the run's `grants`:

```json
{
  "grants": {
    "mail": {
      "cap": "email.resend.send",
      "needs": {
        "http": {
          "cap": "net.http.post",
          "idempotency_key": true
        },
        "key": {
          "env": "RESEND_API_KEY"
        },
        "sender": "you@yourdomain.com"
      }
    }
  }
}
```

## Notes

Grant `http` with `"idempotency_key": true`: Resend deduplicates by Idempotency-Key for 24 hours.

Addresses with capital letters are refused rather than lowercased: the part before `@` may be case-sensitive, and two addresses must never be merged for an `only` check.

`sender` must be an address on a domain verified with Resend.

## Status

Tested end to end against a local server that answers like the service, as part of
the [Staal test suite](https://github.com/laurells/staal/blob/main/tests/packages_test.rs).
Not yet run against the live service.

## License

Private and unlicensed: no permission is granted to use, copy or distribute this
code. A license will be chosen before any public release.
