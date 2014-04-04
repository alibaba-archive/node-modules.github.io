TESTS = test/*.test.js
REPORTER = spec
TIMEOUT = 1000
MOCHA_OPTS =
PWD = `pwd`

install:
	@npm install --registry=http://r.cnpmjs.org --disturl=http://dist.cnpmjs.org

build b: install
	@-rm index.html
	@./node_modules/.bin/gitbook build --output=$(PWD)/book/
	@cp _index.html index.html

serve s: install
	@-rm index.html
	@./node_modules/.bin/gitbook serve --output=$(PWD)/book/
	@cp _index.html index.html

contributors: install
	@./node_modules/.bin/contributors -f plain -o AUTHORS

.PHONY: test
