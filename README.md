# 4_BGT
## Reikalingi instaliavimai
 + Meta Mask Google Chrome extention
 + Ganache

## 1. Verslo modelis

B2C meno įrankių parduotuvė. B2C (bussines to customer) - Verslas parduoda prekes vartotojui, o nuosavybės teisė perduodama po apmokėjimo.
Šiame verslo modelyje dalyvauja:
 + Pardavėjas – parduoda meno įrankius (pieštukus, dažus, drobes ir kt.)
 + Pirkėjas – perka meno įrankius, naudodamasis decentralizuota aplikacija
 + Kurjeris – pristato prekes pirkėjui

Išmaniosios sutarties funkcijos:
 + Automatizuoja užsakymo vykdymą ir mokėjimo procesą
 + Užtikrina, kad prekės bus pristatytos tik gavus apmokėjimą
 + Sekama užsakymo būsena (gamyba, išsiuntimas, pristatymas)
 + Lėšos pervedamos pardavėjui tik po sėkmingo pristatymo.

Logika:
 + Produkto pridėjimas:
   Pardavėjas gali pridėti produktą, suteikti jam pavadinamą ir kainą.
   
 + Produkto "deaktyvavimas":
   Pardavėjas gali ištrinti produktą iš sąrašo pakeičiant jo kainą į 0.

 + Užsakymo pateikimas:
   Pirkėjas pasirenka prekes ir pateikia užsakymą per decentralizuotą aplikaciją.
   Pirkėjas atlieka mokėjimą, o pinigai laikomi išmaniojoje sutartyje kaip užstatas.
   
 + Užsakymo vykdymas:
   Pardavėjas gauna užsakymo informaciją ir pradeda vykdymą.
   Prekės pereina į „Gamybos“ būseną, o vėliau į „Paruošta siuntimui“ būseną.
   
 + Siuntimas:
   Pardavėjas perduoda prekes kurjeriui, o sutartis automatiškai atnaujina užsakymo būseną į „Išsiųsta“.
   
 + Pristatymas:
   Kurjeris pristato prekes pirkėjui ir pažymi užsakymą kaip „Pristatyta“ per decentralizuotą aplikaciją.
   
 + Apmokėjimas pardavėjui:
   Kai kurjeris patvirtina, kad užsakymas pristatytas, išmanioji sutartis automatiškai perveda prekių kainą pardavėjui.
   


   
## 2. Verslo logikos realizacija

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract ArtStore {
    enum OrderStatus { Created, Shipped, Delivered }

    struct Order {
        uint256 orderId;
        address buyer;
        address seller;
        address courier;
        uint256 price;
        OrderStatus status;
    }

    struct Product {
        uint256 productId;
        string name;
        uint256 price;
        address seller;
    }

    mapping(uint256 => Product) public products;
    mapping(uint256 => Order) private orders;
    uint256 public nextProductId;
    uint256 public nextOrderId;

    event ProductAdded(uint256 productId, string name, uint256 price);
    event OrderCreated(uint256 orderId, address buyer, uint256 price);
    event OrderStatusUpdated(uint256 orderId, OrderStatus status);
    event PaymentReleased(uint256 orderId, address seller);

    // Produkto kūrimas
    function addProduct(string calldata name, uint256 price) external {
        require(price > 0, "Price must be greater than zero");

        uint256 productId = nextProductId++;
        products[productId] = Product({
            productId: productId,
            name: name,
            price: price,
            seller: msg.sender 
        });

        emit ProductAdded(productId, name, price);
    }

    // Užsakymo kūrimas
    function createOrder(address seller, address courier, uint256 productId) external payable {
        
        Product memory product = products[productId];
        require(product.price > 0, "Invalid product");
        // Patikrina, ar pirkėjas atlieka mokėjimą
        require(msg.value == product.price, "Incorrect payment amount");
         // Užtikrina, kad pardavėjo adresas yra toks pat kaip prie produkto
        require(product.seller == seller, "Seller address does not match the product seller");
         // Užtikrina, kad kurjerio adresas yra galiojantis
        require(courier != address(0), "Invalid courier address");
         // Sukuriamas unikalus užsakymo ID
        uint256 orderId = nextOrderId++;
        
         // Įrašomas naujas užsakymas į mapping
        orders[orderId] = Order({
            orderId: orderId,
            buyer: msg.sender,
            seller: seller,
            courier: courier,
            price: msg.value,
            status: OrderStatus.Created
        });

        emit OrderCreated(orderId, msg.sender, msg.value);
    }

    // Užsakymo būsenos atnaujinimas į „Shipped“
    function markAsShipped(uint256 orderId) external {
        Order storage order = orders[orderId];
        require(msg.sender == order.seller, "Only seller can mark as shipped");
        require(order.status == OrderStatus.Created, "Order is not in created status");

        order.status = OrderStatus.Shipped;
        emit OrderStatusUpdated(orderId, OrderStatus.Shipped);
    }
 
    // Užsakymo būsenos atnaujinimas į „Delivered“
    function markAsDelivered(uint256 orderId) external {
        Order storage order = orders[orderId];
        require(msg.sender == order.courier, "Only courier can mark as delivered");
        require(order.status == OrderStatus.Shipped, "Order is not shipped");

        order.status = OrderStatus.Delivered;
        emit OrderStatusUpdated(orderId, OrderStatus.Delivered);
        releasePayment(orderId);
    }

    // Mokėjimo išleidimas
    function releasePayment(uint256 orderId) internal {
        Order storage order = orders[orderId];
        require(order.status == OrderStatus.Delivered, "Order must be delivered");
        require(order.price > 0, "No funds to release");

        uint256 payment = order.price;
        order.price = 0; // Apsauga nuo pakartotinio vykdymo

        
        (bool success, ) = payable(order.seller).call{value: payment}("");
        require(success, "Payment failed");

        emit PaymentReleased(orderId, order.seller);
    }

    // Užsakymo informacijos peržiūra
    function getOrder(uint256 orderId) external view returns (Order memory) {
        return orders[orderId];
    }
}

```

## 3. Testavimas
### Lokalus
1. Remix IDE pasirinkau 0.8.26 versiją ir po kompiliavimo kodą rodo, kad kompiliacija sėkminga.
2. Lokalūjį testavimą nusprendžiau daryti su Ganache. Parsisiuntus ir įsijungus programą paspažiau „Quickstart Ethereum“ ir man pakrovė 10 account'ų.
3. Remix IDE nuėjau į „Deploy & run transactions“ ir prie environment pasirinkau „Custom - external http provider“ ir pakeičiau RCP URL galą į 7545, kad sutaptų su Ganache'je nurodytu.
4. Paspaudžiau Deploy
   Čia iškilo problema, nes išmetė pranešimą „Gas estimation failed“. Nors aš ir spausdavau „Cancel transaction“ account'ų skiltyje vistiek rodė Ganache's sukurtus account'us. Pasidomėjau, kodėl iškilo ši problema ir nusprendžiau dar kartą taisyti kodą. Pataisius is supaprastinus šiek tiek kodą ir bandžius deploy'int problema išliko. Taip pat buvo siūlyta  pakeisti versiją su kuria kompiliavau į šiek tiek senesnę, nes problema galėoj kilti dėl evm versijų nesuderinamumo, bet tai taip pat nepadėjo. Kodą tikrinau daugybę kartų ir neradau klaidų ar vietų keliančių problemų, taigi nuspredžiau, jog problema kyla bandant atlikti testavimą, o ne su pačiu kodu. Tuomet išsiaiškinau, kaip suderinti evm versijas. Remix IDE failą sukompiliavau su 0.8.0 versija, o Ganache hardfork nustačiau į Istanbul. Tada vėl atlikus 2-4 punktus.
   
<img  src="https://github.com/user-attachments/assets/fc8df274-2fab-4f78-8abb-b7518013bb6e"  width="200">

Paspaudus „Deploy“ „Debug“ skyrelyje pasirodė blokas.

<img  src="https://github.com/user-attachments/assets/ef38759c-76d0-4e49-9844-cd8bbacacda8"   width="500">

Taip pat Ganache programoje prie transakcijų rodė „Contact creation“, prie blokų taip pat rodė vieną bloką.

<img  src="https://github.com/user-attachments/assets/481060c9-d9b8-4749-bc0c-ec9d774da81b"   width="500">

Prie account'ų galime matyti, kad buvo nuskaičiuota dėl sukūrimo.

<img  src="https://github.com/user-attachments/assets/697dd3c2-f987-4485-a409-0f4da4b71a62"   width="500">

Na o Remix galima matyti funkijas, kurias reikia pratestuoti.

<img  src="https://github.com/user-attachments/assets/7a9cd208-d370-4ff0-8083-87baa9cd6fa2"   width="200">

5. Tikrinau funkciją addProduct

Remix palikau tą pati account'ą, su kuriuo sukūriau konraktą, ir prie produkto info įvedžiau jo pavadinimą ir kainą. 1000000000000000000 Wei = 1 Eth.
   
<img  src="https://github.com/user-attachments/assets/390fbe98-e7e9-4371-8161-0db0afe73b83"  width="200">

„Debug“ skyrelyje pasirodė blokas.

<img  src="https://github.com/user-attachments/assets/dcc832b8-5a29-4e01-b88d-2f215227a139"   width="500">

Buvo vėl nuskaičiuota iš pardavėjo account'o.

<img  src="https://github.com/user-attachments/assets/78f3d71c-f806-4e88-9c53-d67f5d2bc3c9"   width="500">

Atsirado dae viena transakciją.

<img src="https://github.com/user-attachments/assets/2313490f-8b4c-46b4-b533-932041115dff"   width="400">

Remix prie products įvedus produkto kodą galima matyti jo informaciją.

<img  src="https://github.com/user-attachments/assets/876586c2-d7a6-4fe2-af95-50a89ebca6a4"   width="300">

6. Tikrinau funkciją createOrder

Pakeičiau account'ą į kitą ir įvedžiau norimo produkto kainą.

<img  src="https://github.com/user-attachments/assets/bf470014-7b1a-4870-84f1-26699a4e07f2"   width="200">

Taip pat įvedžiau produkto pardavėjo ir kurjerio address ir produkto ID.

<img  src="https://github.com/user-attachments/assets/c58bbdd5-32b8-4168-8a46-61b0cbb5b7da"   width="300">

„Debug“ skyrelyje pasirodė blokas.

<img  src="https://github.com/user-attachments/assets/72ea32aa-f0fa-4364-85de-20673c32e6ed"   width="500">

Kiekis pasikeitė tik pirkėjo account'e.

<img  src="https://github.com/user-attachments/assets/e9eb28dd-b97a-47e8-8cd7-9947bb0f1486"   width="400">

Prisdėjo transakcija sąraše.

<img  src="https://github.com/user-attachments/assets/d65910ca-38ee-4a7d-8f39-7eff2dc14135"   width="500">

Grįžę į Remix „Debug“ prie bloko galima matyti užsakymo info.

<img  src="https://github.com/user-attachments/assets/8534e4dc-79d0-470b-8ce8-c6ee6a55923d"   width="500">

7. Tikrinau funkciją MarkAsShipped

Vėl pakeičiau account'ą į pardavėjo. 

<img  src="https://github.com/user-attachments/assets/e5d03b36-e75d-438c-b2ab-a2de712e3ca1"   width="400">

Įvedžiau OrderID. 

<img  src="https://github.com/user-attachments/assets/f1459c17-b675-48ab-9121-d73259b51f14"   width="300">

Remix „Debug“ prie bloko galima matyti, kad status yra 1, tai reiškia, kad statusas pakeistas į „shipped“.

<img  src="https://github.com/user-attachments/assets/8fc44d86-80fa-440b-bf3a-d7ebb1e374b6"   width="500">

Taip pat pasipildė transakcijų sąrašas.

<img  src="https://github.com/user-attachments/assets/bc3e0aed-23a3-41e0-8a6c-a811cdf18584"   width="500">

8. Tikrinau funkciją MarkAsDelivered

Pakeičiau account'ą į nurodyta kurjerio account'ą createOrder funkcijoje. 

<img  src="https://github.com/user-attachments/assets/41e22d8a-4248-4258-816c-d65a29c60fc5"   width="200">

Įvedžiau OrderID. 

<img  src="https://github.com/user-attachments/assets/6b6220a3-3352-48ae-8d04-29365b1d7ce3"   width="200">

Remix „Debug“ prie bloko galima matyti, kad status yra 2, tai reiškia, kad statusas pakeistas į „delivered“.

<img  src="https://github.com/user-attachments/assets/f3c29b9a-1841-43bf-be2a-9063baeab57d"   width="500">

Tik pardavėjo account'o kiekis pinigų pasikeitė - padidėjo. 

<img  src="https://github.com/user-attachments/assets/80156007-c88c-458a-b296-b9811da95c6a"   width="400">

### Tinklinis

Atlikti tinkliniam testavimui parsisiunčiau MetaMask ir susikūriau piniginę. MetaMask support'e radau pranešimą. 

<img  src="https://github.com/user-attachments/assets/795a0d67-2ae1-44a6-b788-58c277888a64"   width="400">

Po apačia testavimui buvo nurodytas pasiūlymai - Sepolia ir Holešky. Sepolia jau integruota į MetaMask, o Holešky ne. Iš padžių reikėjo padaryti, kad rodytų testinius network. Kilo problemu gaunant testitniu eth.
