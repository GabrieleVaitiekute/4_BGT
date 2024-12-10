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
   
 + Ginčų sprendimas (jei reikalinga):
   Jei pirkėjas nepatenkintas pristatymu (pvz., prekės sugadintos), jis gali inicijuoti ginčo procesą.
   Ginčo metu lėšos lieka užrakintos, kol pasiekiamas sprendimas.

   
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

    mapping(uint256 => Order) private orders;
    uint256 public nextOrderId;

    event OrderCreated(uint256 orderId, address buyer, uint256 price);
    event OrderStatusUpdated(uint256 orderId, OrderStatus status);
    event PaymentReleased(uint256 orderId, address seller);

    // Užsakymo kūrimas
    function createOrder(address seller, address courier) external payable {
        // Patikrina, ar pirkėjas atlieka mokėjimą
        require(msg.value > 0, "Payment required to create order");
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

        payable(order.seller).transfer(payment);

        emit PaymentReleased(orderId, order.seller);
    }

    // Užsakymo informacijos peržiūra
    function getOrder(uint256 orderId) external view returns (Order memory) {
        return orders[orderId];
    }
}
```

## 3. Testavimas

## 4. Logų peržiūra

## 5. Front-end
