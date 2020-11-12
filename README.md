# SolanaSwift

Solana-blockchain client, written in pure swift.

[![Version](https://img.shields.io/cocoapods/v/SolanaSwift.svg?style=flat)](https://cocoapods.org/pods/SolanaSwift)
[![License](https://img.shields.io/cocoapods/l/SolanaSwift.svg?style=flat)](https://www.apache.org/licenses/LICENSE-2.0.html)
[![Platform](https://img.shields.io/cocoapods/p/SolanaSwift.svg?style=flat)](https://cocoapods.org/pods/SolanaSwift)

![Image of Yaktocat](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/b89ca268-71c2-4603-8128-a56c89bf5d14/1200x628_solana_%281%29.jpg?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20201111%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20201111T121728Z&X-Amz-Expires=86400&X-Amz-Signature=ca8faa5e0e3a3deab1f6291d1af3ba7f0361bfd2db23ba3357d81b0fc912bc78&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%221200x628_solana_%281%29.jpg%22)

## Features
- [x] Key pairs generation
- [x] Networking with POST methods for comunicating with solana-based networking system
- [x] Create, sign transactions
- [ ] Socket communication // TODO

## Example

To run the example project, clone the repo, and run `pod install` from the Example directory first.

## Requirements
- iOS 11 or later
- RxSwift

## Dependencies
- RxAlamofire
- TweetNacl
- CryptoSwift
- Socket-io client // TODO

## Installation

SolanaSwift is available through [CocoaPods](https://cocoapods.org). To install
it, simply add the following line to your Podfile:

```ruby
pod 'SolanaSwift'
```

## How to use
* Every class or struct is defined within namespace `SolanaSDK`, for example: `SolanaSDK.Account`, `SolanaSDK.Error`.

* Import
```swift
import SolanaSwift
```

* Create an `AccountStorage` for saving account's `keyPairs` (public and private key), for example: `KeychainAccountStorage` for saving into `Keychain` in production, or `InMemoryAccountStorage` for temporarily saving into memory for testing. The `AccountStorage` must conform to protocol `SolanaSDKAccountStorage`, which has 2 requirements: function for saving `save(_ account:) throws` and computed property `account: SolanaSDK.Account?` for retrieving user's account.

Example:
```swift
import KeychainSwift
struct KeychainAccountStorage: SolanaSDKAccountStorage {
    let tokenKey = <YOUR_KEY_TO_STORE_IN_KEYCHAIN>
    func save(_ account: SolanaSDK.Account) throws {
        let data = try JSONEncoder().encode(account)
        keychain.set(data, forKey: tokenKey)
    }
    
    var account: SolanaSDK.Account? {
        guard let data = keychain.getData(tokenKey) else {return nil}
        return try? JSONDecoder().decode(SolanaSDK.Account.self, from: data)
    }
}

struct InMemoryAccountStorage: SolanaSDKAccountStorage {
    private var _account: SolanaSDK.Account?
    func save(_ account: SolanaSDK.Account) throws {
        _account = account
    }
    var account: SolanaSDK.Account? {
        _account
    }
}
```
* Creating an instance of `SolanaSDK`:
```swift
let solanaSDK = SolanaSDK(endpoint: <YOUR_API_ENDPOINT>, accountStorage: KeychainAccountStorage.shared)
```
* Creating an account:
```swift
let mnemonic = Mnemonic()
let account = try SolanaSDK.Account(phrase: mnemonic.phrase)
try solanaSDK.accountStorage.save(account)
```
* Send pre-defined POST methods, which return a `RxSwift.Single`. [List of predefined methods](https://github.com/p2p-org/solana-swift/blob/main/SolanaSwift/Classes/Generated/SolanaSDK%2BGeneratedMethods.swift):

Example:
```swift
solanaSDK.getBalance(account: account, commitment: "recent")
    .subscribe(onNext: {balance in
        print(balance)
    })
    .disposed(by: disposeBag)
```
* Send token with `send(to:, amount:)` method:
```swift
solanaSDK.send(to: <ACCOUNT_PUBLIC_KEY>, amount: <LAMPORTS>)
    .subscribe(onNext: {result in
        print(result)
    })
    .disposed(by: disposeBag)
```
* Send custom method, which was not defined by using method `request<T: Decodable>(method:, path:, bcMethod:, parameters:) -> Single<T>`

Example:
```swift
(solanaSDK.request(method: .post, bcMethod: "aNewMethodThatReturnsAString", parameters: []) as Single<String>)
```
* Observe socket events:

// TODO: The socket implementation has not been ready yet.

## Contribution
- For supporting new methods, data types, edit `SolanaSDK+Methods` or `SolanaSDK+Models`
- For testing, run `Example` project and creating test using `RxBlocking`
- Welcome to contribute, feel free to change and open a PR.

## Author
Chung Tran, chung.t@p2p.org

## License

SolanaSwift is available under the MIT license. See the LICENSE file for more info.
