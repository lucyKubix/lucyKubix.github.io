# Kubix Developer Documentation

Developer documentation and guides for Kubix developers and client services.

## About

This is the internal documentation site for Kubix, providing guides, best practices, and helpful information for developers and client services. It includes tooling recommendations, VS Code/Cursor extensions, best practices, workflows, and more.

## Development

This site is built with Jekyll using the Cayman theme.

### Prerequisites

- Ruby
- Bundler

### Getting Started

1. Clone the repository
2. Install dependencies:
   ```bash
   bundle install
   ```

3. Build and serve the site locally:
   ```bash
   bundle exec jekyll serve
   ```

4. Visit `http://localhost:4000` in your browser

### Running Tests

Run the test suite to ensure the site builds successfully:

```bash
script/bootstrap  # Run once to install dependencies
script/cibuild    # Run tests
```

## Contributing

To contribute to these docs:

1. Make your changes to the relevant markdown files
2. Test locally to ensure everything builds correctly
3. Submit a pull request

For questions or suggestions about missing content, please contact [Lucy](mailto:lucy.salmons@kubixmedia.co.uk).

## Project Structure

- `developers/` - Developer guides and documentation
- `_includes/` - Reusable Liquid includes
- `_layouts/` - Page layouts
- `_sass/` - Sass stylesheets
- `assets/` - Static assets (CSS, images)

## License

See [LICENSE](LICENSE) file for details.
