# Testing

How to test...

- [an error path with Http and MockBackend?](#how-do-i-test-an-error-path-with-http-and-mockbackend)
- [an input field](#how-to-test-an-input-field)
- [an Observable](#how-to-test-an-observable)



### How do I test an error path with Http and MockBackend?

[A solution from @jgongo](https://gitter.im/angular/angular?at=56a915aa80ad69394a7ad156) with some additional changes based on [comments from @krimple](https://gitter.im/angular/angular?at=56913628d739f50a36029ef7)
```typescript
beforeEachProviders([
 HTTP_PROVIDERS,
 MockBackend,
 provide(XHRBackend, {useClass: MockBackend}),
 AuthenticationService
]);


it('should return an error upon unsuccessful authentication', injectAsync([XHRBackend, AuthenticationService], (mockBackend, authenticationService) => {
 return new Promise((resolve, reject) => {
  mockBackend.connections.subscribe((connection: MockConnection) => {
   connection.readyState = ReadyState.Done;
   connection.response.error(new Response(new ResponseOptions({
   body: {error: 'invalid_grant', error_description: 'Incorrect username and / or password'},
   status: 400
  })));
  });
   
  authenticationService.authenticate({username: 'john.doe', password: '12345'}).subscribe(
   (oauthTokenResponse) => {
    reject(Error('Unexpected successful response'));
   },
   (error: Response) => {
    expect(error.status).toBe(400);
    expect(error.json().error).toBe('invalid_grant');
    expect(error.json().error_description).toBe('Incorrect username and / or password');
   }
  );
 });
}));
```


### How to test an Input field?

```typescript 
@Component({
	selector : 'my-cmp',
	template : ''
})
class MyCmp {
	myFunction = jasmine.createSpy('myFunction');
}

describe('Input', () => {

	it('should dispatch an input event', inject([TestComponentBuilder], (tcb) => {
		let template = '<input type="text" (input)="myFunction()" />';
		tcb.overrideTemplate(MyCmp, template)
			.createAsync(MyCmp)
			.then(fixture => {
				fixture.detectChanges();

				let input = fixture.debugElement.query(By.css('input'));

				input.nativeElement.dispatchEvent(new Event('input'));

				expect(fixture.componentInstance.myFunction).toHaveBeenCalled();
			});
	}));

});
```
Reference : https://github.com/ericmartinezr/ng2-test-samples/blob/master/test-samples/input/input.spec.ts



### How to test an Observable?

```typescript 

describe('Observable', () => {

	it('should work with a raw Observable', (done) => {
		Observable.of(1).toPromise().then((val) => {
			expect(val + 1).toEqual(2);
			done();
		});
	});

	it('should work using injectAsync', injectAsync([], () => {
		return Observable.of(2).toPromise().then(val => {
			expect(val).toEqual(2);
		});
	}));

});
```
Reference : https://github.com/ericmartinezr/ng2-test-samples/blob/master/test-samples/observable/observable.spec.ts