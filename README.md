# blog.frd.mn

Static blog based on [Metalsmith](http://www.metalsmith.io/)

## Installation

1. Make sure you've installed all requirements
2. Clone this repository:  
  `git clone https://github.com/frdmn/blog.frd.mn`
3. Rename `info.json.example` to `info.json` and adjust appropriately
4. Install the project using `npn`, `bower` & `grunt`:  
  `npm install && bower install && grunt`

## Infos

### Front Matter

**For Posts:**

```
---
title: "Install Nginx, PHP-FPM, MySQL and phpMyAdmin on OS X using Homebrew"
slug: "install-nginx-php-fpm-mysql-and-phpmyadmin-on-os-x-using-homebrew" // optional, will be set to title
disqus_id: 13
date: 2014-10-19
---
```

**For Pages:**

```
---
title: Imprint
---
```

## Contributing

1. Fork it
2. Create your feature branch: `git checkout -b feature/my-new-feature`
3. Commit your changes: `git commit -am 'Add some feature'`
4. Push to the branch: `git push origin feature/my-new-feature`
5. Submit a pull request

## Requirements / Dependencies

* NodeJS / npm
* Grunt
* Bower

## Version

1.0.0

## License

[MIT](LICENSE)
