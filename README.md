# Interchain Security

[![Go Report Card](https://goreportcard.com/badge/github.com/cosmos/interchain-security)](https://goreportcard.com/report/github.com/cosmos/interchain-security)
[![Security Rating](https://sonarcloud.io/api/project_badges/measure?project=cosmos_interchain-security&metric=security_rating)](https://sonarcloud.io/summary/new_code?id=cosmos_interchain-security)
[![Vulnerabilities](https://sonarcloud.io/api/project_badges/measure?project=cosmos_interchain-security&metric=vulnerabilities)](https://sonarcloud.io/summary/new_code?id=cosmos_interchain-security)
[![Bugs](https://sonarcloud.io/api/project_badges/measure?project=cosmos_interchain-security&metric=bugs)](https://sonarcloud.io/summary/new_code?id=cosmos_interchain-security)
[![Lines of Code](https://sonarcloud.io/api/project_badges/measure?project=cosmos_interchain-security&metric=ncloc)](https://sonarcloud.io/summary/new_code?id=cosmos_interchain-security)
[![Coverage](https://sonarcloud.io/api/project_badges/measure?project=cosmos_interchain-security&metric=coverage)](https://sonarcloud.io/summary/new_code?id=cosmos_interchain-security)

**Zincirler arası güvenlik (ICS)**, zincirler arası güvenliğin uygulanması için kod barındırır. Repo şu anda bir WIP ve zincirler arası güvenliğin V1'i hedefliyor. Zincirler Arası Güvenlik Protokolü hakkında daha fazla bilgi için [spesifikasyona](https://github.com/cosmos/ibc/blob/main/spec/app/ics-028-cross-chain-validation/README.md) bir göz atın.

CCV, zincirler arası validasyon anlamına gelir ve sağlayıcı ve tüketici blok zincirleri arasındaki staking ve slashing (ceza kesme) iletişimiyle ilgili olarak zincirler arası güvenliğin alt kümesini ifade eder. Sağlayıcı blockchain, tüketici blockchain(ler)'de yapılan değişiklikleri iletirken, tüketici blockchain, sağlayıcı blok zincirine kesintiye uğramış kanıtlar iletebilir.

The code for CCV is housed under [x/ccv](./x/ccv). The `types` folder contains types and related functions that are used by both provider and consumer chains, while the `consumer` module contains the code run by consumer chains and the `provider` module contains the code run by provider chain.

CCV kodu x/ccv altında bulunur. `Tipler` klasörü, hem sağlayıcı hem de tüketici zincirleri tarafından kullanılan tipleri ve ilgili işlevleri içerirken, `tüketici` modülü tüketici zincirleri tarafından çalıştırılan kodu içerir ve `sağlayıcı` (provider) modülü, sağlayıcı zinciri tarafından çalıştırılan kodu içerir.

## Yönergeler

**Önkoşullar**

```bash
## OSX ya da Linux için

# go 1.18 (https://formulae.brew.sh/formula/go)
brew install go@1.18
# jq (opsiyonel, testnet için) (https://formulae.brew.sh/formula/jq)
brew install jq
# docker (opsiyonel, integrasyon testleri için, testnet) (https://docs.docker.com/get-docker/)

```

**Binary Dosyalarını Yükleme ve Çalıştırma**

```bash
# interchain-security-pd ve interchain-security-cd binary dosyalarını yükleme
make install
# sağlayıcıyı çalıştırma
interchain-security-pd
# tüketiciyi çalıştırma
interchain-security-cd
# (Yukarıdakiler başarısız olursa, ~/go/bin $PATH yolunu sağlayın)
export PATH=$PATH:$(go env GOPATH)/bin
```

Merak ediyorsanız [Makefile] (./Makefile) denetleyin.

## Test Etme

### Birim Testleri

Birim testleri basit bağımsız işlevsellik ve CRUD işlemleri için kullanışlıdır. Birim testleri golang'ın standart test paketini kullanmalı ve ```<file being tested>_test.go``` olarak biçimlendirilmiş dosyalarda tanımlanmalıdır.

[Mocked external keepers](./testutil/keeper/mocks.go) ([gomock](https://github.com/golang/mock)) ile uygulanan daha karmaşık işlevselliği test etmek için mevcuttur, ancak yine de yalnızca tek bir düğümdeki yürütme ile ilgilidir. Yani Internode ya da zincirler arası iletişim yok.

### Uçtan Uca (e2e) Testler

[Uçtan uca testler](./e2e-tests/) [IBC test paketini](https://github.com/cosmos/ibc-go/tree/main/testing) kullanır ve bir birim testinden daha geniş olan, ancak yine de bellek içinde valide edilebilir. Yani  Gelişen blokların yararlı olacağı kod, simüle edilmiş handshake'ler, simüle edilmiş paket röleleri vb. 

### Diferansiyel Testler (WIP)

Uçtan uca testler (e2e) testlerine benzer şekilde ancak sistem durumunu bir model uygulamasından üretilen beklenen bir durumla karşılaştırırlar.

### Entegrasyon Testleri

[Entegrasyon testleri](./integration-tests/) bir Docker konteyneri içinde gerçek tüketici ve sağlayıcı zincir binary dosyalarını çalıştırır ve en üst düzey işlevsellik ile ilgilidir. Entegrasyon testleri, kodu sürmek ve doğrulamak için CLI'den çağrılan sorguları/işlemleri kullanır.

### Testleri Çalıştırma

```bash
# make kullanarak tüm statik analiz, birim, E2E ve entegrasyon testlerini çalıştırın
TODO
# make kullanarak tüm birim ve E2E testlerini çalıştırın
make test
# go kullanarak tüm birim ve E2E testlerini çalıştırın
go test ./...
# tüm birim ve E2E testlerini ayrıntılı çıktı ile çalıştırın
go test -v ./..
# tüm birim ve E2E testlerini kapsam istatistikleri ile çalıştırın
go test -cover ./..
# tek bir birim testi çalıştırın
go test -run <unit-test-name> path/to/package
# örnek: tek bir birim testi çalıştırın
go test -run TestSlashAcks ./x/ccv/provider/keeper
# tek bir e2e testi çalıştırın
go test -run <test-suite-name>/<test-name> ./...
# örnek: tek bir e2e testi çalıştırın
go test -run TestProviderTestSuite/TestPacketRoundtrip ./...
# tüm entegrasyon testlerini çalıştırın
go run ./integration-tests/...
# local bir cosmos sdk ile tüm entegrasyon testlerini çalıştırın 
go run ./integration-tests/... --local-sdk-path "/Users/bob/Documents/cosmos-sdk/"
# golang native fuzz testlerini çalıştırın (https://go.dev/doc/tutorial/fuzz)
go test -fuzz=<regex-to-match-test-name>
```

### Linterler ve Statik Analiz

Kodda [CodeQL](https://codeql.github.com/), [SonarCloud](https://sonarcloud.io/), [golangci-lint](https://golangci-lint.run/) ve [gosec](https://github.com/securego/gosec) dahil olmak üzere birkaç analizör kullanılır. Bunlardan bazıları Prs ECT'yi taahhüt ederken GitHub'da çalıştırılır, ancak bazı araçlar da yerel olarak uygulanabilir ve Golang'a yerleştirilir.

```bash
# kodu biçimlendirmek ve basitleştirmek için gofmt (https://pkg.go.dev/cmd/gofmt)
gofmt -w -s -e .
# şüpheli Kodu aramak için go vet (https://pkg.go.dev/cmd/vet)
go vet ./...
```

Bazı kullanışlı araçlar, [pre-commit](https://pre-commit.com/hooks.html) kullanılarak depoya dahil edilir. pre-commit geliştirici araçlarını her git commit ya da `pre-commit run --all-files` ile manuel olarak çalıştırmanıza olanak tanır. Ayrıntılar için bakınız; [config](./.pre-commit-config.yaml). Bu repoda kancalar git'e yüklenmez, çünkü bu hantal olabilir, ancak yine de onlardan yararlanmak mümkündür.

```bash
## Önkoşullar

# pre-commit
brew install pre-commit
# goimports (https://pkg.go.dev/golang.org/x/tools/cmd/goimports)
go install golang.org/x/tools/cmd/goimports@latest
# gocyclo (https://github.com/fzipp/gocyclo)
go install github.com/fzipp/gocyclo/cmd/gocyclo@latest
# go-critic https://github.com/go-critic/go-critic
go install github.com/go-critic/go-critic/cmd/gocritic@latest

## Araşöarı çalıştırma

pre-commit run --all-files
```

### Hata Ayıklama

VSCode kullanıyorsanız, birim testlerinde hata ayıklamak veya binary dosyalara gitmek için bkz. [vscode-go/wiki/debugging](https://github.com/golang/vscode-go/wiki/debugging)

### Daha fazlası

TestNet için zamanında daha fazla talimat eklenecektir.

## Daha Fazlasını Öğren

- [IBC Docs](https://docs.cosmos.network/master/ibc/)
- [IBC Protocol](https://ibcprotocol.org/)
- [IBC Specs](https://github.com/cosmos/ibc)
- [Cosmos SDK documentation](https://docs.cosmos.network)
- [Cosmos SDK Tutorials](https://tutorials.cosmos.network)
- [Discord](https://discord.gg/cosmosnetwork)
