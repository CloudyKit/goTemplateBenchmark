# goTemplateBenchmark
comparing the performance of different template engines

## full featured template engines
* [Ace](https://github.com/yosssi/ace)
* [Amber](https://github.com/eknkc/amber)
* [Go](https://golang.org/pkg/html/template)
* [Handlebars](https://github.com/aymerick/raymond)
* [Kasia](https://github.com/ziutek/kasia.go)
* [Mustache](https://github.com/hoisie/mustache)
* [Pongo2](https://github.com/flosch/pongo2)
* [Soy](https://github.com/robfig/soy)
* [Jet](https://github.com/CloudyKit/jet)

## precompilation to Go code
* [ego](https://github.com/benbjohnson/ego)
* [egon](https://github.com/commondream/egon)
* [egonslinso](https://github.com/SlinSo/egon)
* [ftmpl](https://github.com/tkrajina/ftmpl)
* [Gorazor](https://github.com/sipin/gorazor)
* [Quicktemplate](https://github.com/valyala/quicktemplate)

## transpiling to HTML
* [Damsel](https://github.com/dskinner/damsel)

## Why?
Just for fun. Go Templates work nice out of the box and should be used for rendering from a security point of view.
If you really care about performance you should cache the rendered output.

on second thought:
I have some templates that cannot be cached in my production code, thats why I'm interested in performant
HTML generation using templates. After trying the code generation based projects I liked ego most, but some
features where missing and generated code could be optimized further. That's why I created a fork
and included the results in this benchmark.

## Results
Changed the environment to my local dev laptop: i7-6700T  16GB Mem
Golang: 1.7.1

### full featured template engines
```
go test -bench "k(Ace|Amber|Golang|Handlebars|Kasia|Mustache|Pongo2|Soy|JetHTML)$" -benchmem -benchtime=3s | pb
```
| Name           |      Runs |  µs/op |  B/op | allocations/op |
| --- | --- | --- | --- | --- |
| Ace            |   300,000 | 16.234 | 5,608 |             77 |
| Amber          | 1,000,000 |  5.438 | 1,929 |             39 |
| Golang         | 1,000,000 |  5.366 | 1,904 |             38 |
| Handlebars     |   300,000 | 12.630 | 4,260 |             90 |
| **JetHTML**        | 3,000,000 |  1.441 |   691 |              0 |
| Kasia          | 1,000,000 |  3.194 | 2,147 |             26 |
| Mustache       | 1,000,000 |  4.253 | 1,569 |             28 |
| Pongo2         | 1,000,000 |  4.402 | 3,302 |             46 |
| Soy            | 1,000,000 |  3.108 | 1,863 |             26 |


### precompilation to Go code
```
go test -bench "k(Ego|Egon|EgonSlinso|Quicktemplate|Ftmpl|Gorazor)$" -benchmem -benchtime=3s | pb
```
|  Name              |       Runs | µs/op |  B/op | allocations/op |
| --- | --- | --- | --- | --- |
| Ego               |  5,000,000 | 1.169 |   914 |              8 |
| Egon              |  2,000,000 | 2.290 |   827 |             22 |
| **EgonSlinso**        | 10,000,000 | 0.629 |   828 |              0 |
| Ftmpl             |  3,000,000 | 1.683 | 1,142 |             12 |
| Gorazor           |  3,000,000 | 1.632 |   613 |             11 |
| **Quicktemplate**     | 10,000,000 | 0.452 |   799 |              0 |


### transpiling to HTML
I removed Damsel, because transpilation should just happen once at startup. If you cache the transpilation result, which is recommended, you would have the same performance numbers as html/template for rendering.

### more complex test with template inheritance (if possible)
```
go test . -bench="Complex" -benchmem -benchtime=3s | pb
```
| Name                     |      Runs |  µs/op |   B/op | allocations/op |
| --- | --- | --- | --- | --- |
| ComplexEgo               | 1,000,000 |  5.305 |  2,561 |             41 |
| **ComplexEgoSlinso**         | 2,000,000 |  2.346 |  2,070 |              7 |
| ComplexEgon              |   300,000 | 10.206 |  4,792 |            101 |
| ComplexFtmpl             |   500,000 |  8.487 |  5,300 |             48 |
| ComplexFtmplInclude      |   500,000 |  9.522 |  5,300 |             48 |
| ComplexGolang            |   100,000 | 44.813 | 12,415 |            295 |
| ComplexGorazor           |   300,000 | 13.306 |  8,327 |             73 |
| ComplexJetHTML           |   500,000 | 10.537 |  4,164 |              5 |
| ComplexMustache          |   200,000 | 27.743 |  7,856 |            166 |
| **ComplexQuicktemplate**     | 2,000,000 |  2.610 |  1,892 |              0 |

## Security
All packages assume that template authors are trusted. If you allow custom templates you have to sanitize your user input e.g. [bluemonday](https://github.com/microcosm-cc/bluemonday). Generally speaking I would suggest to sanitize every input not just HTML-input. 

| Framework | Security | Comment |
| --------- | -------- | ------- |
| Ace | No | |
| amber | No | |
| Damsel | Yes, if html/template is used for execution | Damsel transpiles to HTML |
| ego | Partial (html.EscapeString) | only HTML, others need to be called manually |
| egon | Partial (html.EscapeString) | only HTML, others need to be called manually |
| egonslinso | Partial (html.EscapeString) | only HTML, others need to be called manually |
| ftmpl | Partial (html.EscapeString) | only HTML, others need to be called manually |
| Go | Yes | contextual escaping [html/template Security Model](https://golang.org/pkg/html/template/#hdr-Security_Model) |
| Gorazor | Partial (template.HTMLEscapeString) | only HTML, others need to be called manually |
| Handlebars | Partial (raymond.escape) | only HTML |
| Jet | Partial (html.EscapeString) | Autoescape for HTML, others need to be called manually |
| Kasia | Partial (kasia.WriteEscapedHtml) | only HTML |
| Mustache | Partial (template.HTMLEscape) | only HTML |
| Pongo2 | Partial (pongo2.filterEscape, pongo2.filterEscapejs) | autoescape only escapes HTML, others could be implemented as pongo filters |
| Quicktemplate | Partial (html.EscapeString) | only HTML, others need to be called manually |
| Soy | Partial (template.HTMLEscapeString, url.QueryEscape, template.JSEscapeString) | autoescape only escapes HTML, contextual escaping is defined as a project goal |
