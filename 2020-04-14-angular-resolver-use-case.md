Resolvers in Angular - Example
==============================
Resolver is an interface that class can implement to be a data provider. 
That data provider class is used by Angular's router to provide the data during navigation.
That interface defines one method, *resolve()*. It's invoked on navigation start event.
Important thing to note here is that router will wait with executing navigation until data requested for resolver is ready.

I found Resolvers useful when there's a component displaying data in the same way, but the data itself needs to be different based on path.

## Resolver - Example of usage
Lets take reporting system for example.
Consider that system has possibility to create & share reports by administrators.
They're shared as read-only.
Person that has access to report should see the same data as owner of that report.

So we have two situations:
- Owner viewing report that he has created
- Other person viewing shared report

Those two use cases could be solved by two different API endpoints (that return the data based on different business rules).
- Reports/__id__
- Reports/Shared/__id__

And this is where Resolvers are handy.
They can provide different data to component, using different api endpoints. Based on route.

```javascript

const routes: Routes = [
  {
    path: ':id',
    component: ReportsComponent,
    resolve: { report: ReportResolver }
  },
  {
    path: 'shared/:id',
    component: ReportsComponent,
    resolve: { report: SharedReportResolver }
  }
];

```

Report resolver would be implemented like this

```javascript

export class ReportResolver implements Resolve<ReportResolved> {

    constructor(private reportsProvider: ReportsProvider) { }

    resolve(route: ActivatedRouteSnapshot, state: RouterStateSnapshot): Observable<ReportResolved> {
        return this.reportsProvider.getReport(route.data.id)
            .pipe(
                map(report => ({ report: report })),
                catchError(error => {
                    /* Handle the error */
                })
            );
    }
}

```

Shared report would look pretty much the same, but would call other service method.

### Summary

This is one way of handling that problem.
Another way could be checking current route by ActivatedRoute service in ngOnInit. Based on that get data that you want. 
But I find this solution more elegant, because:
- data is ready just in the moment that page is loaded 
- simplified component's code by using built in feature
- no need `if` statements to handle different situations
- component stays dumb.

