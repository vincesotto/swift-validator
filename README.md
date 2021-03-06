Swift-Validator
===============

Swift Validator is a rule-based validation library for Swift.

![Swift Validator](https://dl.dropboxusercontent.com/u/18055521/swift-validator.gif)

## Core Concepts

* ```UITextField``` + ```ValidationRule``` go into  ```Validator```
* ```UITextField``` + ```ValidationError``` come out of ```Validator```
* ```UITextField``` is registered to ```Validator```
* ```Validator``` evaluates ```ValidationRules``` sequentially and stops evaluating when a ```ValidationRule``` fails. 
* Keys are used to allow field registration in TableViewControllers and complex view hierarchies

## Quick Start

Initialize the ```Validator``` by setting a delegate to a View Controller or other object.

```swift

// ViewController.swift

let validator = Validator()

override func viewDidLoad() {
    super.viewDidLoad()
}

```

Register the fields that you want to validate

```swift

var fields:[String] = ["FullName", "Email", "Phone"]

// Validation Rules are evaluated from left to right. The first rule is ValidationRuleType.Required the second is ValidationRuleType.FullName.
validator.registerFieldByKey(fields[0], textField:nameTextField, rules: [.Required, .FullName])
validator.registerFieldByKey(fields[1], textField:emailTextField, rules: [.Required, .Email])
validator.registerFieldByKe(fields[2], textField:phoneTextField, rules: [.Required, .PhoneNumber])

```

Validate Individual Field

```swift

validator.validateFieldByKey(fields[0], delegate:self)

// ValidationFieldDelegate methods
func validationFieldSuccess(key:String, validField:UITextField){
	validField.backgroundColor = UIColor.greenColor()
}

func validationFieldFailure(key:String, error:ValidationError){
	println(error.error.description)
}

```

Validate All Fields

```swift

validator.validateAllKeys(delegate:self)

// ValidationDelegate methods

func validationWasSuccessful() {
	// submit the form
}

func validationFailed(errors:[String:ValidationError]){
	// turn the fields to red
	for error in errors.values {
		error.textField.backgroundColor = UIColor.redColor()
		println("error -> \(error.error.description)")
	}
}

```

## Custom Validation 

We will create a ```SSNValidation``` class to show how to create your own Validation. A United States Social Security Number (or SSN) is a field that consists of XXX-XX-XXXX. 

Create a class that implements the Validation protocol

```swift

class SSNValidation: Validation {
    let SSN_REGEX = "^\\d{3}-\\d{2}-\\d{4}$"
    
    func validate(value: String) -> (Bool, ValidationErrorType) {
        if let ssnTest = NSPredicate(format: "SELF MATCHES %@", SSN_REGEX) {
            if ssnTest.evaluateWithObject(value) {
                return (true, .NoError)
            }
            return (false, .SocialSecurity) // We will create this later ValidationErrorType.SocialSecurity
        }
        return (false, .SocialSecurity)
    }
    
}

```

Add the ```ValidationRuleType.SocialSecurity``` 

```swift

enum ValidationRuleType {
    case Required,
    Email,
    Password,
    MinLength,
    MaxLength,
    ZipCode,
    PhoneNumber,
    FullName,
    SocialSecurity	// Added to the ValidationRuleTypes
}

```

Add the ```ValidationErrorType.SocialSecurity``` and ```description()```

```swift

enum ValidationErrorType {
    case Required,
    Email,
    Password,
    MinLength,
    MaxLength,
    ZipCode,
    PhoneNumber,
    FullName,
    SocialSecurity,	// Added to the ValidationErrorTypes
    NoError
    
    func description() -> String {
        switch self {
        case .Required:
            return "Required field"
        case .Email:
            return "Must be a valid email"
        case .MaxLength:
            return "This field should be less than"
        case .ZipCode:
            return "5 digit zipcode"
        case .PhoneNumber:
            return "10 digit phone number"
        case .Password:
            return "Must be at least 8 characters"
        case .FullName:
            return "Provide a first & last name"
        // Adding the desired error message
        case .SocialSecurity:
        	return "SSN is XXX-XX-XXXX"
        default:
            return ""
        }
    }
    
}

```
Register the ```SSNValidation``` with the ```ValidationFactory```

```swift

class ValidationFactory {
    class func validationForRule(rule:ValidationRuleType) -> Validation {
        switch rule {
        case .Required:
            return RequiredValidation()
        case .Email:
            return EmailValidation()
        case .MinLength:
            return MinLengthValidation()
        case .MaxLength:
            return MaxLengthValidation()
        case .PhoneNumber:
            return PhoneNumberValidation()
        case .ZipCode:
            return ZipCodeValidation()
        case .FullName:
            return FullNameValidation()
        // Add Validation to allow Factory to create one on the fly for you
        case .SocialSecurity:
        	return SSNValidation()
        default:
            return RequiredValidation()
        }
    }
}

```
Credits
-------

Swift Validator is written and maintained by Jeff Potter [@jpotts18](http://twitter.com/jpotts18) and friends.

Currently funded and maintained by [RingSeven](http://ringseven.com)

![RingSeven](https://avatars1.githubusercontent.com/u/8309133?v=3&s=200)

## Contributing

1. [Fork it](https://github.com/jpotts18/swift-validator/fork)
2. Create your feature branch `git checkout -b my-new-feature`
3. Commit your changes `git commit -am 'Add some feature'`
4. Push to the branch `git push origin my-new-feature`
5. Create a new Pull Request
