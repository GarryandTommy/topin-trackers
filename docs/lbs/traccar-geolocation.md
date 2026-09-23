# Traccar cell geolocation

> **Use at your own risk.** External geolocation services have their own credentials, quotas, terms and data handling. Never publish API keys.

The ZX909 decoder work produces usable cellular identifiers inside Traccar. It does **not** itself convert them into coordinates. A separate geolocation provider performs that step.

During project testing, geolocation was intentionally disabled while the LTE decoder was known to be wrong. A later test configuration used Unwired Labs. This documents the tested setup only and is not a requirement or provider recommendation.

Public examples should use placeholders such as `YOUR_GEOLOCATION_API_KEY`.

When validating LBS, retain raw cell identifiers separately from provider-returned coordinates so decoder errors can be distinguished from provider/database errors.
