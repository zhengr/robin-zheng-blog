---
title: Hyperledger区块链技术催促供应链（SCM）再度进化，真正实现供应链全局可视化
date: 2017-09-15 00:00:00
author: robin zheng
cover: true
img: https://raw.githubusercontent.com/zhengr/zhengr.github.io/master/assets/images/20170915Capture16.PNG
categories: 区块链
tags:
  - Hyperledger
  - Blockchain
  - Fabric
  - Supply chain
  - Inventory
---
## 区块链技术日趋成熟，看未来五年，供应链将迎来一次巨大的升级。这是一次性打通供应链上下游的机会，信息不再是由点对点数据传输实现共享和传递。

区块链能够围绕核心企业甚至终端消费者搭建一条包括制造商、供应商、零售商、物流公司、消费者在内的信息联盟，并将资金流、信息流、事务流都记录在这链条上。区块链的核心之处在于，参与的各方必须沿着时间轴实时提交事务数据，信息一旦经过背书写入区块，就只能读取，不能随意更改、删除，伪造。

接下来我将基于hyperledger composer团队的Vehicle Lifecycle Chain Demo和大家分享区块链如何支撑一辆车从订单到生产再到报废的整个生命周期，同时车管所或者交警队等监管机构如何全程监管整个商业过程。

### 设定业务场景

这个区块链网络跟踪从订单->制造->报废的车辆的生命周期，包括私人所有者、制造商、车管所/交警队和报废厂。车管所/交警队能够在整个过程中提供监督。

一个私人客户`PriavteOwner` 参与者将通过制造商的应用程序提交`PlaceOrder`（下单）事务。制造商`Manufacturer`确认客户订单后将提交`UpdateOrderStatus` 事务，通过该事务可以根据车辆生产的进度不断更新订单状态（PLACED -> SCHEDULED_FOR_MANUFACTURE -> VIN_ASSIGNED -> OWNER_ASSIGNED -> DELIVERED），以便客户可以随时获知进度。车辆制造到一定阶段，制造商也将通过提交一个`ApplicationForVehicleRegistrationCertificate` 的事务来申请注册证书。当车辆被交付客户后，私人客户可以提交`PrivateVehicleTransfer`事务以完成客户间车辆买卖。监管机构可以对整个流程进行监督，并提交`UpdateSuspicious` 事务，以查看任何可能违反监管规定的可疑车辆。车辆报废厂可以提交一个`ScrapVehicle`或`ScrapAllVehiclesByColour`事务，来完成车辆的生命周期。

![20170915Capture16](https://raw.githubusercontent.com/zhengr/zhengr.github.io/master/assets/images/20170915Capture16.PNG)

这个区块链网络中需要做如下定义:

**Participants**`AuctionHouse` `Company` `Manufacturer` `PrivateOwner` `Regulator` `ScrapMerchant`

**Assets**`Order` `Vehicle`

**Transactions**`PlaceOrder` `UpdateOrderStatus` `ApplicationForVehicleRegistrationCertificate` `PrivateVehicleTransfer` `ScrapVehicle` `UpdateSuspicious` `ScrapAllVehiclesByColour` `SetupDemo`

**Events**`PlaceOrderEvent` `UpdateOrderStatusEvent` `ScrapVehicleEvent`

![20170915Capture1](https://raw.githubusercontent.com/zhengr/zhengr.github.io/master/assets/images/20170915Capture1.PNG)

我们接下来逐一对以上这些元素展开介绍， 首先我们将构建五个models，

> * [参考：网络定义](https://hyperledger.github.io/composer/business-network/businessnetworkdefinition.html) - Business Network Definition

- models/base.cto
- models/business.cto
- models/manufacturer.cto
- models/vda.cto
- models/vehicle.cto


```
/* models/base.cto
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 * http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

/**
  * A library of standard reusable types
  */
namespace composer.base

enum Gender {
 o MALE
 o FEMALE
 o OTHER
}

abstract participant Person {
  o String title optional
  o String firstName  optional
  o String lastName  optional
  o String[] middleNames  optional
  o Gender gender optional
  o String[] nationalities optional
  o ContactDetails contactDetails optional
  o BirthDetails birthDetails optional
  o DeathDetails deathDetails optional
}

concept ContactDetails {
  o String email  optional
  o String mobilePhone  optional
  o String homePhone optional
  o Address address optional
}

concept BirthDetails {
  o DateTime dateOfBirth optional
  o String placeOfBirth optional
}

concept DeathDetails {
  o DateTime dateOfDeath optional
  o String placeOfDeath optional
}

/**
 * A concept for a simple street address
 */
concept Address {
  o String city optional
  o String country optional
  o String locality optional
  o String region optional
  o String street optional
  o String street2 optional
  o String street3 optional
  o String postalCode optional
  o String postOfficeBoxNumber optional
}
```

```
/* models/business.cto
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 * http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
namespace composer.business

import composer.base.Person
import composer.base.Address

/**
 * An abstract participant type in this business network
 */
abstract participant Business {
  o Address headquarters optional
  o String name optional
  --> Manager[] managers optional
}

abstract participant Employee extends Person {
  --> Business employer
  --> Manager manager optional
  o DateTime startDate optional
  o String employmentStatus optional
  o String department optional
  o String jobRole optional
}

abstract participant Manager extends Employee {
  --> Employee[] directReports optional
}
```

```
/* models/manufacturer.cto
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 * http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

/**
 * Order model for vehicle lifecycle
 */
namespace org.acme.vehicle.lifecycle.manufacturer

import org.vda.VehicleDetails
import composer.business.Business
import composer.base.Person


/**
 * Status of an order
 */
enum OrderStatus {
  o PLACED
  o SCHEDULED_FOR_MANUFACTURE
  o VIN_ASSIGNED
  o OWNER_ASSIGNED
  o DELIVERED
}

/**
 * A manufacturer of vehicles
 */
participant Manufacturer identified by companyId extends Business {
  o String companyId
}

/**
 * An order for a vehicle to be fulfilled by a manufacturer
 * and dispatched to an orderer (Person).
 */
asset Order identified by orderId {
  o String orderId
  o VehicleDetails vehicleDetails
  o OrderStatus orderStatus
  --> Manufacturer manufacturer
  --> Person orderer
  o UpdateOrderStatus[] statusUpdates optional //TODO (LG): Unit test this
}

/**
 * Transaction to create an order
 */
transaction PlaceOrder {
  o String orderId
  o VehicleDetails vehicleDetails
  --> Manufacturer manufacturer
  --> Person orderer
}

event PlaceOrderEvent {
  o String orderId
  o VehicleDetails vehicleDetails
}

/**
 * Transaction to update the status of an order
 */
transaction UpdateOrderStatus{
  o OrderStatus orderStatus
  o String vin optional
  o String v5c optional
  o String numberPlate optional
  --> Order order
}

event UpdateOrderStatusEvent {
  o OrderStatus orderStatus
  o Order order
}
```

```json
/* models/vda.cto
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 * http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
/**
 * Model based on the DVLA vehicle definition and registration process
 */
namespace org.vda
import composer.base.Person
/**
 * Based on DVLA document V355/1
 */
enum TaxClass {
  o PRIVATE_LIGHT_GOODS
  o PETROL_CAR
  o DIESEL_CAR
  o ALTERNATIVE_FUEL_CAR
  o LIGHT_GOODS_VEHICLE
  o EURO4_LIGHT_GOODS_VEHICLE
  o EURO5_LIGHT_GOODS_VEHICLE
  o HEAVY_GOODS_VEHICLE
  o PRIVATE_HEAVY_GOODS_VEHICLE
  o SPECIAL_TYPES
  o HAULAGE_VEHICLES
  o BUS
  o MOTORCYCLE
  o ELECTRIC_MOTOCYCLE
  o SPECIAL_VEHICLES
  o SMALL_ISLAND_VEHICLES
  o RECOVERY_VEHICLE
  o SPECIAL_CONCESSIONARY
  o EMERGENCY_VEHICLE
  o EXCEMPT_VEHICLE
}
concept VehicleDetails {
  o String make
  o String modelType
  o String colour
  o String vin optional
  o String v5c optional
  o String numberPlate optional
  o String modelVariant optional
  o String modelVersion optional
  o String bodyType optional
  o TaxClass taxationClass optional
  o Integer revenueWeight optional
  o Integer cylinderCapacity optional
  o Double co2 optional // g/km
  o String typeOfFuel optional
  o Integer numberOfSeats optional
  o Integer numberOfStandingPlaces optional
  o String wheelPlan optional
  o String vehicleCategory optional
  o String typeApprovalNumber optional
  o Double maxNetPower optional // kW
  o String engineNumber optional
  o Double maxPermissibleMass optional
  o Double massInService optional
  o Double powerWeightRatio optional
  o TrailerDetails trailerDetails optional
  o SoundDetails soundDetails optional
  o ExhaustEmissions exhaustEmissions optional
}
concept TrailerDetails {
  o Double maxPermissibleTowableMassBraked
  o Double maxPermissibleTowableMassUnbraked
}
concept SoundDetails {
  o Double stationary
  o Double engineSpeed
  o Double driveBy
}
concept ExhaustEmissions {
  o Double co
  o Double hc
  o Double nox
  o Double hc_plus_nox
  o Double particulates
}
concept VehicleTransferLogEntry {
  --> Vehicle vehicle
  --> Person buyer
  --> Person seller optional
  o DateTime timestamp
}
/**
 * Based on the DVLA V62 document
 */
transaction ApplicationForVehicleRegistrationCertificate {
  o VehicleDetails vehicleDetails
  --> Person keeper
  o String dvlaFleetNumber optional
  o String driversLicenseNumber optional
  o Long mileage optional
  o String previousPostCode optional
}
abstract transaction VehicleTransaction  {
  --> Vehicle vehicle
}
/**
 * DVLA V5C
 */
transaction PrivateVehicleTransfer extends VehicleTransaction {
  --> Person seller
  --> Person buyer
  o String specialNotes optional
}
enum VehicleStatus {
  o ACTIVE
  o OFF_THE_ROAD
  o SCRAPPED
}
asset Vehicle identified by vin {
  o String vin
  o VehicleDetails vehicleDetails
  o VehicleStatus vehicleStatus
  --> Person owner optional
  o String numberPlate optional
  o String suspiciousMessage optional
  o VehicleTransferLogEntry[] logEntries optional
}
transaction ScrapVehicle extends VehicleTransaction {
  o VehicleTransaction[] logEntries optional
}
transaction UpdateSuspicious extends VehicleTransaction {
  o String message
}
transaction ScrapAllVehiclesByColour  {
  o String colour
}
event ScrapVehicleEvent {
  --> Vehicle vehicle
}
```

```json
/* models/vehicle.cto
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 * http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

/**
 * Vehicle model for vehicle lifecycle
 */
namespace org.acme.vehicle.lifecycle

import composer.base.Person
import composer.business.Business

participant PrivateOwner identified by email extends Person {
  o String email
}

participant Company identified by companyId extends Business {
  o String companyId
}

participant Regulator extends Company {
}

participant AuctionHouse extends Company {
}

participant ScrapMerchant extends Company {
}

transaction SetupDemo  {
}

```

在访问安全控制Access Control里我们赋予涉众全部访问权 Permissions.acl

```json
/**
 * Sample access control list.
 */
rule Default {
    description: "Allow all participants access to all resources"
    participant: "ANY"
    operation: ALL
    resource: "org.acme.vehicle.lifecycle.**"
    action: ALLOW
}

rule SystemACL {
  description:  "System ACL to permit all access"
  participant: "org.hyperledger.composer.system.Participant"
  operation: ALL
  resource: "org.hyperledger.composer.system.**"
  action: ALLOW
}
```

然后我们开始编写业务逻辑， 也就是事务处理逻辑部分。

- lib/setup.js
- lib/manufacturer.js
- lib/vda.js

setup.js 是用来初始化用例数据，其他的事务处理接口都可以从以下scripts文件中找到对应的代码，具体的业务逻辑实现可以详研代码。

```javascript
/* setup.js 
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 * http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

/**
 * Setup the demo
 * @param {org.acme.vehicle.lifecycle.SetupDemo} setupDemo - the SetupDemo transaction
 * @transaction
 */
function setupDemo(setupDemo) {
    console.log('setupDemo');

    var factory = getFactory();
    var NS_M = 'org.acme.vehicle.lifecycle.manufacturer';
    var NS = 'org.acme.vehicle.lifecycle';
    var NS_D = 'org.vda';

    var names = ['zhengr', 'simon', 'jake', 'anastasia', 'matthew', 'mark', 'fenglian', 'sam', 'james', 'nick', 'caroline', 'rachel', 'john', 'rob', 'tom', 'paul', 'ed', 'dave', 'anthony', 'toby', 'ant', 'matt', 'anna'];
    var vehicles = {
        'Arium': {
            'Nova': [
                {
                    'vin': '156478954',
                    'colour': 'white',
                    'vehicleStatus': 'ACTIVE'
                }
            ],
            'Nebula': [
                {
                    'vin': '652345894',
                    'colour': 'blue',
                    'vehicleStatus': 'ACTIVE'
                }
            ]
        }, 
        'Morde': {
            'Putt': [
                {
                    'vin': '6437956437', 
                    'colour': 'black',
                    'vehicleStatus': 'ACTIVE', 
                    'suspiciousMessage': 'Mileage anomaly'
                },
                {
                    'vin': '857642213', 
                    'colour': 'red',
                    'vehicleStatus': 'ACTIVE'
                },
                {
                    'vin': '542376495', 
                    'colour': 'silver',
                    'vehicleStatus': 'ACTIVE'
                }
            ],
            'Pluto': [
                {
                    'vin': '976431649', 
                    'colour': 'white',
                    'vehicleStatus': 'ACTIVE'
                },
                {
                    'vin': '564215468', 
                    'colour': 'green',
                    'vehicleStatus': 'ACTIVE', 
                    'suspiciousMessage': 'Insurance write-off but still active'
                },
                {
                    'vin': '784512464', 
                    'colour': 'grey',
                    'vehicleStatus': 'ACTIVE'
                }
            ]
        },
        'Ridge': {
            'Cannon': [
                {
                    'vin': '457645764',
                    'colour': 'red',
                    'vehicleStatus': 'ACTIVE'
                },
                {
                    'vin': '312457645',
                    'colour': 'white',
                    'vehicleStatus': 'ACTIVE', 
                    'suspiciousMessage': 'Suspicious ownership sequence'
                },
                {
                    'vin': '65235647',
                    'colour': 'silver',
                    'vehicleStatus': 'ACTIVE', 
                    'suspiciousMessage': 'Untaxed vehicle'
                }
            ], 
            'Rancher': [
                {
                    'vin': '85654575',
                    'colour': 'blue',
                    'vehicleStatus': 'ACTIVE'
                }, 
                {
                    'vin': '326548754',
                    'colour': 'white',
                    'vehicleStatus': 'ACTIVE', 
                    'suspiciousMessage': 'Uninsured vehicle'
                }
            ]
        }
    };
    
    var manufacturers = [];
    var privateOwners = [];

    for (var name in vehicles) {
        var manufacturer = factory.newResource(NS_M, 'Manufacturer', name);
        manufacturers.push(manufacturer);
    }

   for(var i=0; i<names.length; i++) {
       var name = names[i];
       var privateOwner = factory.newResource(NS, 'PrivateOwner', name);
       privateOwners.push(privateOwner);
   }

    var regulator = factory.newResource(NS, 'Regulator', 'regulator');


    var privateOwnerRegistry;
    var vehicleRegistry;

    return getParticipantRegistry(NS + '.Regulator')
        .then(function(regulatorRegistry) {
            return regulatorRegistry.add(regulator);
        })
        .then(function() {
            return getParticipantRegistry(NS_M + '.Manufacturer');
        })
        .then(function(manufacturerRegistry) {
            return manufacturerRegistry.addAll(manufacturers);
        })
        .then(function() {
            return getParticipantRegistry(NS + '.PrivateOwner');
        })
        .then(function(privateOwnerRegistry) {
            return privateOwnerRegistry.addAll(privateOwners);
        })
        .then(function() {
            return getAssetRegistry(NS_D + '.Vehicle');
        })
        .then(function(vehicleRegistry) {
            var vs = [];
            var carCount = 0;
            for (var mName in vehicles) {
                var manufacturer = vehicles[mName];
                for (var mModel in manufacturer) {
                    var model = manufacturer[mModel];
                    for(var i=0; i<model.length; i++) {
                        var vehicleTemplate = model[i];
                        var vehicle = factory.newResource(NS_D, 'Vehicle', vehicleTemplate.vin);
                        vehicle.owner = factory.newRelationship(NS, 'PrivateOwner', names[carCount]);
                        vehicle.vehicleStatus = vehicleTemplate.vehicleStatus;
                        vehicle.vehicleDetails = factory.newConcept(NS_D, 'VehicleDetails');
                        vehicle.vehicleDetails.make = mName; 
                        vehicle.vehicleDetails.modelType = mModel; 
                        vehicle.vehicleDetails.colour = vehicleTemplate.colour; 
                        vehicle.vehicleDetails.vin = vehicleTemplate.vin;

                        if (vehicleTemplate.suspiciousMessage) {
                            vehicle.suspiciousMessage = vehicleTemplate.suspiciousMessage;
                        }

                        if (!vehicle.logEntries) {
                            vehicle.logEntries = [];
                        }

                        var logEntry = factory.newConcept(NS_D, 'VehicleTransferLogEntry');
                        logEntry.vehicle = factory.newRelationship(NS_D, 'Vehicle', vehicleTemplate.vin);
                        logEntry.buyer = factory.newRelationship(NS, 'PrivateOwner', names[carCount]);
                        logEntry.timestamp = setupDemo.timestamp

                        vehicle.logEntries.push(logEntry);

                        vs.push(vehicle);
                        carCount++;
                    }
                }
            }
            return vehicleRegistry.addAll(vs);
        });
}
```

```javascript
/* lib/manufacturer.js 
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 * http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */


/**
 * Place an order for a vehicle
 * @param {org.acme.vehicle.lifecycle.manufacturer.PlaceOrder} placeOrder - the PlaceOrder transaction
 * @transaction
 */
function placeOrder(placeOrder) {
    console.log('placeOrder');

    var factory = getFactory();
    var NS_M = 'org.acme.vehicle.lifecycle.manufacturer';
    var NS = 'org.acme.vehicle.lifecycle';
    var NS_D = 'org.vda';

    var order = factory.newResource(NS_M, 'Order', placeOrder.orderId);
    order.vehicleDetails = placeOrder.vehicleDetails;
    order.orderStatus = 'PLACED';
    order.manufacturer = placeOrder.manufacturer;
    order.orderer = factory.newRelationship(NS, 'PrivateOwner', placeOrder.orderer.getIdentifier());

    // save the order
    return getAssetRegistry(order.getFullyQualifiedType())
        .then(function (registry) {
            return registry.add(order);
        })
        .then(function(){
    		var placeOrderEvent = factory.newEvent(NS_M, 'PlaceOrderEvent');
      		placeOrderEvent.orderId = order.orderId;
      		placeOrderEvent.vehicleDetails = order.vehicleDetails;
    		emit(placeOrderEvent);
    	});
}

/**
 * Update the status of an order
 * @param {org.acme.vehicle.lifecycle.manufacturer.UpdateOrderStatus} updateOrderStatus - the UpdateOrderStatus transaction
 * @transaction
 */
function updateOrderStatus(updateOrderStatus) {
    console.log('updateOrderStatus');

    var factory = getFactory();
    var NS_M = 'org.acme.vehicle.lifecycle.manufacturer';
    var NS = 'org.acme.vehicle.lifecycle';
    var NS_D = 'org.vda';

    // save the new status of the order
    updateOrderStatus.order.orderStatus = updateOrderStatus.orderStatus;

  	// get vehicle registry
  	return getAssetRegistry(NS_D + '.Vehicle')
  		.then(function(registry) {
      		if (updateOrderStatus.orderStatus === 'VIN_ASSIGNED') {
            	var vehicle = factory.newResource(NS_D, 'Vehicle', updateOrderStatus.vin );
                vehicle.vehicleDetails = updateOrderStatus.order.vehicleDetails;
                vehicle.vehicleDetails.vin = updateOrderStatus.vin;
                vehicle.vehicleStatus = 'OFF_THE_ROAD';
                return registry.add(vehicle);
            } else if(updateOrderStatus.orderStatus === 'OWNER_ASSIGNED') {
                if (!updateOrderStatus.order.orderer.vehicles) {
                    updateOrderStatus.order.orderer.vehicles = [];
                }

            	return registry.get(updateOrderStatus.vin)
                    .then(function(vehicle) {
                        vehicle.vehicleStatus = 'ACTIVE';
                        vehicle.owner = factory.newRelationship('org.acme.vehicle.lifecycle', 'PrivateOwner', updateOrderStatus.order.orderer.email);
                        vehicle.numberPlate = updateOrderStatus.numberPlate || '';
                        vehicle.vehicleDetails.numberPlate = updateOrderStatus.numberPlate || '';
                        vehicle.vehicleDetails.v5c = updateOrderStatus.v5c || '';
                        if (!vehicle.logEntries) {
                            vehicle.logEntries = [];
                        }
                        var logEntry = factory.newConcept(NS_D, 'VehicleTransferLogEntry');
                        logEntry.vehicle = factory.newRelationship(NS_D, 'Vehicle', updateOrderStatus.vin);
                        logEntry.buyer = factory.newRelationship(NS, 'PrivateOwner', updateOrderStatus.order.orderer.email);
                        logEntry.timestamp = updateOrderStatus.timestamp;
                        vehicle.logEntries.push(logEntry);
                        return registry.update(vehicle);
                    });
            }
    	})
  		.then(function() {
      		// get order registry
    		return getAssetRegistry(updateOrderStatus.order.getFullyQualifiedType());
    	})
  		.then(function(registry) {
      		// update order status
            updateOrderStatus.order.vehicleDetails.vin = updateOrderStatus.vin || '';
            
            if (!updateOrderStatus.order.statusUpdates) {
                updateOrderStatus.order.statusUpdates = [];
            }

            updateOrderStatus.order.statusUpdates.push(updateOrderStatus);

      		return registry.update(updateOrderStatus.order);
    	})
        .then(function(){
    		var updateOrderStatusEvent = factory.newEvent(NS_M, 'UpdateOrderStatusEvent');
      		updateOrderStatusEvent.orderStatus = updateOrderStatus.order.orderStatus;
      		updateOrderStatusEvent.order = updateOrderStatus.order;
    		emit(updateOrderStatusEvent);
    	});
        
}
```

```javascript
/* lib/vda.js
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 * http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */


/**
 * Transfer a vehicle to another private owner
 * @param {org.vda.PrivateVehicleTransfer} privateVehicleTransfer - the PrivateVehicleTransfer transaction
 * @transaction
 */
function privateVehicleTransfer(privateVehicleTransfer) {
    console.log('privateVehicleTransfer');

    var currentParticipant = getCurrentParticipant();


    var NS_M = 'org.acme.vehicle.lifecycle.manufacturer';
    var NS = 'org.acme.vehicle.lifecycle';
    var NS_D = 'org.vda';
    var factory = getFactory();

    var seller = privateVehicleTransfer.seller;
    var buyer = privateVehicleTransfer.buyer;
    var vehicle = privateVehicleTransfer.vehicle;

    //change vehicle owner
    vehicle.owner = buyer;

    //PrivateVehicleTransaction for log
    var vehicleTransferLogEntry = factory.newConcept(NS_D, 'VehicleTransferLogEntry');
    vehicleTransferLogEntry.vehicle = factory.newRelationship(NS_D, 'Vehicle', vehicle.getIdentifier());
    vehicleTransferLogEntry.seller = factory.newRelationship(NS, 'PrivateOwner', seller.getIdentifier());
    vehicleTransferLogEntry.buyer = factory.newRelationship(NS, 'PrivateOwner', buyer.getIdentifier());
    vehicleTransferLogEntry.timestamp = privateVehicleTransfer.timestamp;
    if (!vehicle.logEntries) {
        vehicle.logEntries = [];
    }

    vehicle.logEntries.push(vehicleTransferLogEntry);

    return getAssetRegistry(vehicle.getFullyQualifiedType())
        .then(function(ar) {
            return ar.update(vehicle);
        });
}

/**
 * Scrap a vehicle
 * @param {org.vda.ScrapVehicle} scrapVehicle - the ScrapVehicle transaction
 * @transaction
 */
function scrapVehicle(scrapVehicle) {
    console.log('scrapVehicle');

    var NS_D = 'org.vda';
    var assetRegistry;

    return getAssetRegistry(NS_D + '.Vehicle')
        .then(function(ar) {
            assetRegistry = ar;
            return assetRegistry.get(scrapVehicle.vehicle.getIdentifier());
        })
        .then(function(vehicle){
            vehicle.vehicleStatus = 'SCRAPPED';
            return assetRegistry.update(vehicle);
        });
}

/**
 * Scrap a vehicle
 * @param {org.vda.ScrapAllVehiclesByColour} scrapAllVehicles - the ScrapAllVehicles transaction
 * @transaction
 */
function scrapAllVehiclesByColour(scrapAllVehicles) {
    console.log('scrapVehicle');

    var NS_D = 'org.vda';
    var assetRegistry;

    // create the query
    var q = {
        selector: {
            'vehicleDetails.colour': scrapAllVehicles.colour
        }
    };

    return getAssetRegistry(NS_D + '.Vehicle')
        .then(function (ar){
            assetRegistry = ar;
            return queryNative(JSON.stringify(q));
        })
        .then(function (resultArray) {
            console.log('TP function received query result: ', JSON.stringify(resultArray));
            if (resultArray.length < 1 ) {
                throw new Error('No vehicles found with ' + scrapAllVehicles.colour, resultArray.length);
            }

            var factory = getFactory();
            var promises =[];
            var serializer = getSerializer();
            for (var x = 0; x < resultArray.length; x++) {
                var currentResult = resultArray[x];
                var vehicle = serializer.fromJSON(currentResult.Record);

                vehicle.vehicleStatus = 'SCRAPPED';
                var scrapVehicleEvent = factory.newEvent(NS_D, 'ScrapVehicleEvent');
                scrapVehicleEvent.vehicle = vehicle;
                emit(scrapVehicleEvent);
                promises.push(assetRegistry.update(vehicle));

            }
            return Promise.all(promises);
        });
}
```

通过以上构建， 我们已经完成一个区块链的业务场景构架， 接下来就可以通过调用各个事务接口完成整个业务场景。

首先初始化演示数据

![20170915Capture2](https://raw.githubusercontent.com/zhengr/zhengr.github.io/master/assets/images/20170915Capture2.PNG)

演示数据成功初始化, 这个事务处理将注册三个汽车生产商 `Manufacturer` , 二十三个私人拥有者 `PrivateOwner` 以及一个监管单位 `Regulator` . 同时会有十三辆 `Vehicle` 资产被注册.

![20170915Capture3](https://raw.githubusercontent.com/zhengr/zhengr.github.io/master/assets/images/20170915Capture3.PNG)

首先用户zhengr提交`PlaceOrder`事务处理下订单， 订单号"SO87200315"

```
{
  "$class": "org.acme.vehicle.lifecycle.manufacturer.PlaceOrder",
  "orderId": "SO87200315",
  "vehicleDetails": {
    "$class": "org.vda.VehicleDetails",
    "make": "Arium",
    "modelType": "Gamora",
    "colour": "Sunburst Orange"
  },
  "manufacturer": "resource:org.acme.vehicle.lifecycle.manufacturer.Manufacturer#Arium",
  "orderer": "resource:org.acme.vehicle.lifecycle.PrivateOwner#zhengr"
}
```

![20170915Capture4](https://raw.githubusercontent.com/zhengr/zhengr.github.io/master/assets/images/20170915Capture4.PNG)

![20170915Capture6](https://raw.githubusercontent.com/zhengr/zhengr.github.io/master/assets/images/20170915Capture6.PNG)

这个时候，汽车生产厂商"Arium"从区块链上获得订单信息，并更新订单状态为

![20170915Capture7](https://raw.githubusercontent.com/zhengr/zhengr.github.io/master/assets/images/20170915Capture7.PNG)

![20170915Capture8](https://raw.githubusercontent.com/zhengr/zhengr.github.io/master/assets/images/20170915Capture8.PNG)

可以看到订单信息已经被生产厂商Arium更新，客户zhengr可以很容易从区块链获得信息看到他预定的汽车已经排产。

```
{
  "$class": "org.acme.vehicle.lifecycle.manufacturer.Order",
  "orderId": "SO87200315",
  "vehicleDetails": {
    "$class": "org.vda.VehicleDetails",
    "make": "Arium",
    "modelType": "Gamora",
    "colour": "Sunburst Orange",
    "vin": ""
  },
  "orderStatus": "SCHEDULED_FOR_MANUFACTURE",
  "manufacturer": "resource:org.acme.vehicle.lifecycle.manufacturer.Manufacturer#Arium",
  "orderer": "resource:org.acme.vehicle.lifecycle.PrivateOwner#zhengr",
  "statusUpdates": [
    {
      "$class": "org.acme.vehicle.lifecycle.manufacturer.UpdateOrderStatus",
      "orderStatus": "SCHEDULED_FOR_MANUFACTURE",
      "order": "resource:org.acme.vehicle.lifecycle.manufacturer.Order#SO87200315",
      "transactionId": "4a307c97-4f70-4405-ad0e-7e604c7a3715",
      "timestamp": "2017-09-15T19:20:07.066Z"
    }
  ]
}
```

汽车生产商赋予VIN编号，提交更新订单事务

```
{
  "$class": "org.acme.vehicle.lifecycle.manufacturer.UpdateOrderStatus",
  "orderStatus": "VIN_ASSIGNED",
  "vin": "23538390",
  "v5c": "",
  "numberPlate": "",
  "order": "resource:org.acme.vehicle.lifecycle.manufacturer.Order#SO87200315"
}

```

就会发现"vin": "23538390" 这辆车出现在资产Vehicle的列表中

![20170915Capture9](https://raw.githubusercontent.com/zhengr/zhengr.github.io/master/assets/images/20170915Capture9.PNG)

再次提交订单更新事务处理，可以看到车辆已经被分配给zhengr

```
{
  "$class": "org.acme.vehicle.lifecycle.manufacturer.UpdateOrderStatus",
  "orderStatus": "OWNER_ASSIGNED",
  "vin": "23538390",
  "v5c": "22356707",
  "numberPlate": "JN887323",
  "order": "resource:org.acme.vehicle.lifecycle.manufacturer.Order#SO87200315"
}
```

![20170915Capture10](https://raw.githubusercontent.com/zhengr/zhengr.github.io/master/assets/images/20170915Capture10.PNG)

再次提交订单更新事务，将订单更新为‘DELIVERED’ 已交付状态。至此汽车已经从制造商交付给了客户。

```
{
  "$class": "org.acme.vehicle.lifecycle.manufacturer.UpdateOrderStatus",
  "orderStatus": "DELIVERED",
  "vin": "23538390",
  "v5c": "22356707",
  "numberPlate": "JN887323",
  "order": "resource:org.acme.vehicle.lifecycle.manufacturer.Order#SO87200315"
}
```

![20170915Capture11](https://raw.githubusercontent.com/zhengr/zhengr.github.io/master/assets/images/20170915Capture11.PNG)

zhengr把车转手给了Anna，需要提交一个PrivateVehicleTransfer事务

```
{
  "$class": "org.vda.PrivateVehicleTransfer",
  "seller": "resource:org.acme.vehicle.lifecycle.PrivateOwner#zhengr",
  "buyer": "resource:org.acme.vehicle.lifecycle.PrivateOwner#anna",
  "specialNotes": "Please pay me by bitcoin",
  "vehicle": "resource:org.vda.Vehicle#23538390"
}
```

![20170915Capture12](https://raw.githubusercontent.com/zhengr/zhengr.github.io/master/assets/images/20170915Capture12.PNG)

可以看到车主从zhengr变为了Anna

![20170915Capture14](https://raw.githubusercontent.com/zhengr/zhengr.github.io/master/assets/images/20170915Capture14.PNG)

监管部门可以随时通过监管软件提交UpdateSuspicious事务更新车辆监管信息，报废厂也可以通过提交ScrapVehicle事务报废制定的车辆， 这些操作就不一一赘述了。

```
{
  "$class": "org.vda.UpdateSuspicious",
  "message": "排放超标",
  "vehicle": "resource:org.vda.Vehicle#23538390"
}
```

通过以上的一系列操作可以看出，不同的软件， 不同的公司，不同的人都在同一个链条上对区块进行操作，这样的事务记录是去中心的，提交点不同但是都需要遵守区块链的交易规则。

当一个特定的区块链网络（Hyperledger Fabric V1）成立，相关的ERP、CRM、MES、APP通过相应SDK， REST API，订阅/发布 Event, 都可以非常顺畅的加入到区块链中。当前可以测试通过的系统有：

- Oracle e-business suites R12 
- Oracle Fusion application
- Siebel
- SAP 待测试  

==================补充==========================

本例中车辆制造商的ERP/MES可以是这个样子:

![20170915Capture17](https://raw.githubusercontent.com/zhengr/zhengr.github.io/master/assets/images/20170915Capture17.PNG)

监管机构的可以是这样的系统：

![20170915Capture18](https://raw.githubusercontent.com/zhengr/zhengr.github.io/master/assets/images/20170915Capture18.PNG)

用户可以随时通过手机客户端查看车辆交付进展：

![20170915Capture19](https://raw.githubusercontent.com/zhengr/zhengr.github.io/master/assets/images/20170915Capture19.PNG)
