# How LBS positioning works

LBS positioning uses cellular network information as an alternative or supplement to GNSS.

For the confirmed ZX909 `0x1A` layout, LTE data includes a four-byte TAC, unsigned four-byte ECI and one-byte signal value. These identify network infrastructure; they are not coordinates.

Conceptually:

`ZX909 -> Topin cell data -> Traccar decoder -> cell geolocation provider -> estimated position`

Cell-based location is not equivalent to GNSS. Accuracy depends on the provider database, coverage and available observations. Correctly decoded identifiers can still yield no result or a coarse estimate.

Using an external provider sends network identifiers to that service, so its terms and privacy implications should be considered.
