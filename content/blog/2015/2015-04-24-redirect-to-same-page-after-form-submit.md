---
date: "2015-04-24"
draft: false
title: "Redirect to same page after form submit"
tags: ["form"]
type: "blog"
slug: "redirect-to-same-page-after-form-submit"
author: "Tomáš Votruba"
---

We're on a protected page and we need to force the user to sign in. We have this place all over the application we want the user to get back to where he started after signing in.

Exactly that is solved by [storeRequest](http://api.nette.org/2.0/source-Application.UI.Presenter.php.html#1115) and [restoreRequest](http://api.nette.org/2.0/source-Application.UI.Presenter.php.html#1139), which allows us to "restore request". In these cases you won't have to think about how to redirect back with `$this->redirect(...)`.

You can have a look at working example in [CD-collection](https://github.com/nette/examples/tree/master/CD-collection/app/presenters) in Nette Framework distribution. Specifically

* [`storeRequest()`](https://github.com/nette/examples/blob/master/CD-collection/app/presenters/DashboardPresenter.php#L26)
* [`restoreRequest()`](https://github.com/nette/examples/blob/master/CD-collection/app/presenters/SignPresenter.php#L41)

Lets examplain really quick how these work. Method `UI\Presenter::storeRequest()` stores current request from user with all its parametrs and returns textual key, that representes it, and can be used for invoking the request again.

`DashboardPresenter` handles few actions, that edit some data. So on it's startup, we will check if the user is logged in. If not, we will redirect him to `SignPresenter`.

```php
class DashboardPresenter extends BasePresenter
{
	protected function startup()
	{
		parent::startup();

		if (!$this->user->isLoggedIn()) {
			if ($this->user->logoutReason === Nette\Http\UserStorage::INACTIVITY) {
				$this->flashMessage('You have been signed out due to inactivity. Please sign in again.');
			}
			$this->redirect('Sign:in', array('backlink' => $this->storeRequest()));
		}
	}

	// ...
}
```

`SignPresenter` contains login form and it's handler. Also wery important persistent parameter `$backlink`, which will automatically persist in the url when the user will submit the login form.

```php
class SignPresenter extends BasePresenter
{
	/** @persistent */
	public $backlink = '';

	protected function createComponentSignInForm()
	{
		$form = new Nette\Application\UI\Form;
		// políčka formuláře ...
		$form->onSuccess[] = $this->signInFormSubmitted;
		return $form;
	}

	public function signInFormSubmitted($form)
	{
		// přihlášení ...
		$this->restoreRequest($this->backlink);
		$this->redirect('Dashboard:');
	}

	// ...
}
```

Method `UI\Presenter::restoreRequest()` accepts request key, that we've previously stored. If there is no stored request for the given key, it will do nothing and the "default redirect" will take care.
