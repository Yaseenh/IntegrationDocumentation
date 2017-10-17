

## Create An Action
```csharp
private Action CreateAction ()
{
  return new Action
  {
    Id = //string Required: Action reference prefixed with Client Id,
    ClientId = //string Required: Client Id,
    Name = //string: Name given to this action,
    Reference = //string Required: Line-of-business generated reference number for this action,
    Nature = //anything Required: Valid values: "Unknown" "Product" "Service",
    IsCod = //boolean Required: This is true if cash on delivery else false,
    VolumetricMass = //double : VolumetricMass of the item,
    Volume = //double: Volume of the item,
    Weight = //double: Weight of the item,
    Pieces = //int: Number of pieces,
    Unit = //string: Unit of measure,
    Instructions = //string: Specify any special (delivery) instuctions, pertaining to this action,
    InternalReference = //string: Internal reference number for workflow i.e. picking slip, order number, sales doc etc,
    CustomerReference = //string: Reference number the customer uses to internally refer to this specific action,
    CustomerCode = //string: A reference to the customer,
    ExpectedDelivery = //string: ISO 8601 DateTime string, representing expected delivery of action,
    ActionTypeId = //string: The Trackmatic Id assosiated with the Action Type Name. This is the branch id, suffixed with the ActionTypeId,
    ActionTypeName = //string: Describes what action type this action is i.e. collection, delivery, ctc etc.,
    Pallets = //int: Number of pallets,
    AmountIncl = //double: Number of the amount including vat,
    AmountEx = //double: Number of the amount excluding vat,
    CreatedOn = //string: ISO 8601 DateTime string, representing when action was captured in line-of-business ERP system,
    Direction = //anything: Valid values: "Outbound" "Inbound",
    HandlingUnitIds = //string List: If using handling units this is a list of each of the Id's,

    Owner = new ActionOwner // Owner of this particular action
    { 
      Name = //string: Name of the owner,
      Reference = //string: Reference of the owner,
      Contact = //string: Contact detail of the owner,
    }
  }
}
```
## Create A handling Unit
```csharp
private HandlingUnit CreateHandlingUnit()
{
  return new HandlingUnit
  {
    Id = //string Required: Route reference prefixed with ClientId,
    Barcode = //string Required: Barcode of item,
    CustomerReference = //string: Reference to a customer,
    InternalReference = //string: internal Reference,
    Quantity = //int: Quantity,
    Weight = //double: Weight,
    Status = //anything: pending,

    Dimensions = new HandlingUnitDimensions()//Dimensions of the item
    {
      Height =//double: Height,
      Length =//double:Length,
      Width = //double: Width,
    }
  };
}
```
## Create An Entity
```csharp
private Entity CreateEntity ()
{
  return new Entity
  {
    Id = //string Required: Entity reference prefixed with Client Id,
    Name = //string Required: Name of the entity,
    Reference = //string Required: Line-of-business reference for a particular entity,

    Contacts = new ObservableCollection<EntityContact>
    { 
      new EntityContact
       {  
        Id = //string: ID of the contact,
        FirstName = //string: FirstName of the contact,
        LastName = //string: LastName of the contact,
        TelNo = //string: Telephone Number of the contact,
        CellNo = //string: Cell Number of the contact,
        Email = //string: Email of the contact,
        WorkNo = //string: Work Number of the contact,
        PositionHeld = //string: Position of the contact,
       }
    }

    Requirements = new ObservableCollection<EntityRequirement>() //Used to determine which entity requirements apply to this entity
    { 
          new RequireActionDebrief(),// Verifying each transaction
          new RequireSignature(),// Sign on glass debrief
          new RequireActionImageDebrief(),//
          new RequireCodDebrief(),// Cash on Delivery debrief
          new RequireHandlingUnitDebrief(),// If using handling units - handling unit debrief
          new RequireImageCapture(),// Image capture debrief
          new RequireRatingSimple(),// Good/Bad rated debrief
          new RequireRatingSmiley(),// Simley rated debrief
          new RequireRatingStar()// Star rated debrief
    }
  }
}
```
## Create A Deco
```csharp
private OLocation CreateDeco ()
{
  return new OLocation
  {
    Id = //string Required: It must take the following form, [client id]/[address id]. When the location is adhoc it should take the following form [client id]/$tmp/[address id].,
    Name = //string Required: Name of the deco,
    ClientId = //string Required: Client Id,
    IsActive = //boolean: true to show on routing

    Entrance = new OCoord()
    {
      Latitude = //double: Latitude,
      Longitude = //double: Longitude
    }

    Coords = new SpecializedObservableCollection<OCoord>()
    {
      new OCoord()
       {
         Latitude = //double: Latitude,
         Longitude = //double: Longitude,
         Radius = //double: Radius
       }
    },

    StructuredAddress = new StructuredAddress()
     {
       UnitNo//string: Unit Number,
       BuildingName = //string: Building Name,
       StreetNo = //string: Street Number,
       SubDivisionNumber = //string: Subdivision Number,
       Street = //string: Street,
       Suburb = //string: Suburb,
       City = //string: City,
       Province = //string: Province,
       PostalCode = //string: Postal Code,
       MapCode = //string: Map Code,
     }
  }
}
```
## Create A Route
```csharp
private RouteModel CreateRoute()
{
  return new RouteModel
  {
    Id = //string Required: Route reference prefixed with ClientId,
    Reference = //string Required: Route reference,
    Name = //string Required: Name of Route,
    ClientId = //string Required: Client Id,
    Registration = //string: Vehicle registration,
    StartDate = //string: ISO DateTime string, representing route’s scheduled start date and time,
    TemplateId = //string Required: ISO DateTime string, representing route’s scheduled start time,
    Schedule = //boolean: true 
  };
}
```
## Create the Relationship
```csharp
private UploadModel CreateRelationship()
{
  var model = new UploadModel();
  model.Route = CreateRoute();
  var location = CreateDeco();
  var entity = CreateEntity();
  var action = CreateAction();

  model.Add(location);
  model.Add(entity);
  model.Add(action);

  var relationship = Relationship
                    .Link(action)
                    .To(entity)
                    .At(location);
  model.Add(relationship);

  model = CreateHandlingUnit(); // Add if handling units are been used

  return model;
}
```
## Uploading the transfomed data
```csharp
private void ProcessPipeline()
{
  var pipeline = new CreatePipeline();
  var uploadModel = new UploadModel();
  var transformedModel = uploadModel.Transform();
  var element = new UploadElement (ClientId, transformedModel);
  pipeline.Add(element);
  Dispatch(pipeline);
}
```

# Multiple Addresses
* Note if there are multiple addresses, replace the above "CreateEntity" method with "CreateShipTo" and "CreateSellTo" methods. Also update "CreateRelationship" method to the one below.
```csharp
private Entity CreateShipTo()
{
  return new Entity
    {
    Id = //string Required: It must take the following form when there is multiple addreses [client id]/[Reference]/[address id].,
    Name = //string Required: Name of the entity,
    Reference = //string Required: Line-of-business reference for a particular entity,

    Contacts = new ObservableCollection<EntityContact>
    { 
      new EntityContact
       {  
        Id = //string: Name of the contact,
        FirstName = //string: FirstName of the contact,
        LastName = //string: LastName of the contact,
        TelNo = //string: Telephone Number of the contact,
        CellNo = //string: Cell Number of the contact,
        Email = //string: Email of the contact,
        WorkNo = //string: Work Number of the contact,
        PositionHeld = //string: Position of the contact,
       }
    }

    Requirements = new ObservableCollection<EntityRequirement>() //Used to determine which entity requirements apply to this entity
    { 
          new RequireActionDebrief(),// Verifying each transaction
          new RequireSignature(),// Sign on glass debrief
          new RequireActionImageDebrief(),//
          new RequireCodDebrief(),// Cash on Delivery debrief
          new RequireHandlingUnitDebrief(),// If using handling units - handling unit debrief
          new RequireImageCapture(),// Image capture debrief
          new RequireRatingSimple(),// Good/Bad rated debrief
          new RequireRatingSmiley(),// Simley rated debrief
          new RequireRatingStar()// Star rated debrief
    }
}
```

```csharp
private Entity CreateSellTo()
{
   return new Entity
    {
      Id = //string Required: Entity reference prefixed with Client Id,
      Reference = //string Required: Reference of the Sell to Entity,
      Name = //string Required: Name of the Sell to Entity,
    };
}
```

```csharp
private UploadModel CreateRelationship()
{
  var model = new UploadModel();
  var route = CreateGroupRoute();
  var action = CreateGroupAction();
  var deco = CreateGroupDeco();
  var shipTo = CreateGroupShipTo();
  var sellTo = CreateGroupSellTo();

  model.Add(action);
  model.Add(deco);
  model.Add(shipTo);
  model.Add(sellTo);
  model.Route = route;

  var relationship = Relationship
                    .Link(action)
                    .To(shipTo)
                    .At(deco)
                    .SellTo(sellTo);
  model.Add(relationship);

  model = CreateHandlingUnit(); // Add if handling units are been used

  return model;
}
```