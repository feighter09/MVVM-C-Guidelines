# 0. Architecture

The project follows MVVM, coupled with coordinators at the top level. [This article](https://medium.com/@giovannyorozco24/mvvm-and-coordinator-pattern-together-8920fc0f1f55) goes through this in detail with example code, and the project roughly follows the diagram included:

![architecture diagram](https://cdn-images-1.medium.com/max/2000/1*5nfF7o3WNQvTPttNsi2yEQ.png)

However, the `Model` section is typically broken down into: `Interactors`, `Data Stores`, and `Entities`

Thus, the tests we write are typically formulaic:

## Coordinators
- `.start()` returns the `UIViewController` subclass you expect
- When transition actions occur, `UIViewController`s are presented or dismissed

Example from `MonitorCoordinatorSpec.swift`
```
describe("start()") {
  var viewController: UIViewController!
  beforeEach { viewController = subject.start() }

  it("returns a HomeViewController") {
    expect(viewController).to(beAKindOf(HomeViewController.self))
  }
  
  context("when the settings button is tapped") {
    beforeEach { viewController.viewModel.showSettings() }
    
    it("shows the settings page") {
      expect(viewController.presentedViewController).toEventually(beAKindOf(SetupViewController.self))
    }
  }
}
```

## View Controllers
- Changes in the `ViewModel` update views on screen
    - eg: `UILabel` text updated, `UITableView` cells added/deleted, loading indicator visible, etc
- UI actions call methods on the `ViewModel`

Example from `PhoneHomeViewControllerSpec.swift`
```
describe("the title text") {
  context("when viewModel.titleText is changed") {
    beforeEach { self.viewModel.titleText.value = "new title" }
    
    it("updates the title label") {
      expect(subject.titleLabel.text).to(equal("new title"))
    }
  }
}

describe("tapping the confirm button") {
  beforeEach { subject.confirmButton.sendActions(for: .touchUpInside) }

  it("calls viewModel.confirm") {
    expect(self.viewModel.confirmCalled).to(beTrue())
  }
}
```

## View Models
- Updates in the underlying model trigger changes in formatted data
- Methods called change the underlying model

Example from `PhoneHomeCheckInViewModelSpec.swift`
```
describe("titleText") {
  context("when entering a phone number") {
    beforeEach { subject.result.value = .phone(Country.test, "") }
    
    it("shows 'enter phone number' text") {
      expect(subject.titleText.value).to(equal("YOUR PHONE NUMBER IS YOUR CLAIM TICKET"))
    }
  }
  
  context("when entering a tag") {
    beforeEach { subject.result.value = .tag("") }
    
    it("shows 'enter tag number' text") {
      expect(subject.titleText.value).to(equal("ADD TAG WITH NUMBER"))
    }
  }
}

describe("clear()") {
  beforeEach { subject.clear() }
  
  it("calls numberPadViewModel.clear()") {
    expect(self.numberPadViewModel.clearCalled).to(beTrue())
  }
}
```

## Model

Classes and structs in the model layer end up being more case-by-case, but tend to follow the same format, and are broken into the following subgroups:

### Interactor (Service Objects)
- Encapsulate an action into a nice-to-use API
    - eg: `Login.login(email:, password:)` does:
        - Fetch list of venues
        - Select specific venue
        - Register device to venue
        - Save venue settings
        - Save theme
    - eg: `MoveConveyorToSlot.move(conveyor: to slot:)` does:
        - Make network request with the proper slot params
        - Return errors in a friendly way (`.tagNotInRange`, `.unauthorized`)


### Data Store 
- Perform actions needed to CRUD entities
    - eg: `NetworkClient` performs network requests
    - eg: `UserItemStore` persists the checked-in items and provides API for fetching

### Entities
- The model `struct`s
- The only tests around them should be JSON serialization/deserialization
