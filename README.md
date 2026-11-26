# Shrike

Shrike is an open source web application firewall. A small daemon (`shriked`)
inspects requests handed to it by thin connectors inside web servers such as
nginx, and pluggable engines (machine learning, ModSecurity/CRS rules) decide
whether to allow or block them.

Goals: install one package and get sensible protection with the default
configuration; easy to run for both AI agents and people.

Status: early design. Nothing here is usable yet.

Licensed under the [Apache License 2.0](LICENSE).
