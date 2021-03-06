toolbox
=======

Middleware toolbox provides health chcek, pprof, profile and statistic services for [Macaron](https://gitea.com/macaron/macaron).

[![Build Status](https://drone.gitea.com/api/badges/macaron/toolbox/status.svg)](https://drone.gitea.com/macaron/toolbox) 
[API Reference](https://gowalker.org/gitea.com/macaron/toolbox)

### Installation

	go get gitea.com/macaron/toolbox
	
## Usage

```go
// main.go
import (
	"gitea.com/macaron/macaron"
	"gitea.com/macaron/toolbox"
)

func main() {
  	m := macaron.Classic()
  	m.Use(toolbox.Toolboxer(m))
	m.Run()
}
```

Open your browser and visit `http://localhost:4000/debug` to see the effects.

## Options

`toolbox.Toolboxer` comes with a variety of configuration options:

```go
type dummyChecker struct {
}

func (dc *dummyChecker) Desc() string {
	return "Dummy checker"
}

func (dc *dummyChecker) Check() error {
	return nil
}

// ...
m.Use(toolbox.Toolboxer(m, toolbox.Options{
	URLPrefix:			"/debug",			// URL prefix for toolbox dashboard
	HealthCheckURL:		"/healthcheck", 	// URL for health check request
	HealthCheckers: []HealthChecker{
		new(dummyChecker),
	},										// Health checkers
	HealthCheckFuncs: []*toolbox.HealthCheckFuncDesc{
		&toolbox.HealthCheckFuncDesc{
			Desc: "Database connection",
			Func: func() error { return "OK" },
		},
	},										// Health check functions
	DisableDebug:		false,				// Turns off all debug functionality when true
	PprofURLPrefix:		"/debug/pprof/", 	// URL prefix of pprof
	ProfileURLPrefix:	"/debug/profile/", 	// URL prefix of profile
	ProfilePath:		"profile",			// Path store profile files
}))
// ...
```

## Route Statistic

Toolbox also comes with a route call statistic functionality:

```go
import (
	"os"
	"time"
	//...
	"gitea.com/macaron/toolbox"
)

func main() {
	//...
	m.Get("/", func(t toolbox.Toolbox) {
		start := time.Now()
		
		// Other operations.
		
		t.AddStatistics("GET", "/", time.Since(start))
	})
	
	m.Get("/dump", func(t toolbox.Toolbox) {
		t.GetMap(os.Stdout)
	})
}
```

Output take from test:

```
+---------------------------------------------------+------------+------------------+------------------+------------------+------------------+------------------+
| Request URL                                       | Method     | Times            | Total Used(s)    | Max Used(??s)     | Min Used(??s)     | Avg Used(??s)     |
+---------------------------------------------------+------------+------------------+------------------+------------------+------------------+------------------+
| /api/user                                         | POST       |                2 |         0.000122 |       120.000000 |         2.000000 |        61.000000 |
| /api/user                                         | GET        |                1 |         0.000013 |        13.000000 |        13.000000 |        13.000000 |
| /api/user                                         | DELETE     |                1 |         0.000001 |         1.400000 |         1.400000 |         1.400000 |
| /api/admin                                        | POST       |                1 |         0.000014 |        14.000000 |        14.000000 |        14.000000 |
| /api/user/unknwon                                 | POST       |                1 |         0.000012 |        12.000000 |        12.000000 |        12.000000 |
+---------------------------------------------------+------------+------------------+------------------+------------------+------------------+------------------+
```

## License

This project is under Apache v2 License. See the [LICENSE](LICENSE) file for the full license text.