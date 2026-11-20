# Changelog

All notable changes to this deployment definition are documented in this file.
The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [Unreleased]

### Changed

- Documentation: README rewritten and icon assets added.

## [0.0.1]

### Added

- Initial App-as-Code service definition for PowerDNS-Admin with a PowerDNS Authoritative
  Server (MySQL backend) and a MariaDB 10.11 dependency.
- Web interface on port 8080 (published as 8882) with optional Traefik routing and automatic DNS.
- DNS listener bound to the host LAN address only.
- README with configuration reference and project icon.
