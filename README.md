# 4_BGT
## Reikalingi instaliavimai
 + Truffle IDE (Node ir npm tinkamas versijas jau turiu)
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
         // Užtikrina, kad pardavėjo adresas yra galiojantis
        require(seller != address(0), "Invalid seller address");
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
   Čia iškilo problema, nes išmetė pranešimą „Gas estimation failed“. Nors aš ir spausdavau „Cancel transaction“ account'ų skiltyje vistiek rodė Ganache's sukurtus account'us. Pasidomėjau, kodėl iškilo ši problema ir nusprendžiau dar kartą taisyti kodą. Pataisius is supaprastinus šiek tiek kodą ir bandžius deploy'int problema išliko. Taip pat buvo siūlyta  pakeisti versiją su kuria kompiliavau į šiek tiek senesnę, nes problema galėoj kilti dėl evm versijų nesuderinamumo, bet tai taip pat nepadėjo. Kodą tikrinau daugybę kartų ir neradau klaidų ar vietų keliančių problemų, taigi nuspredžiau, jog problema kyla bandant atlikti testavimą, o ne su pačiu kodu.
   Lokalaus testavimo atlikti nepavyko, todėl nusprendžiau, kad geriau nebandyti ir testuoti tinkle ir atlikti sekančių užduočių.
## 4. Logų peržiūra

## 5. Front-end
