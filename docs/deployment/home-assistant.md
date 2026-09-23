# Home Assistant deployment

> **Use at your own risk.** Replacing server binaries or add-on configuration can stop tracking. Keep backups and a rollback path.

The project runs a local derivative of the official Traccar Home Assistant add-on named **Traccar home54**. It preserves the add-on's MariaDB, nginx and Home Assistant service logic and replaces only `tracker-server.jar` with the locally patched build.

The local `Traccar home54` build runs the relevant Topin regression tests before assembling the patched server package.

The tested deployment uses `topin.port = 5093`.

A previous configuration issue showed that XML entries inside a commented example block can appear in text searches while being ignored by the XML parser. Verify active XML structure, not grep output alone.

The official add-on is retained but stopped while the local add-on is active, providing a rollback route.

Do not publish production credentials, API keys, device identifiers or private network details.
