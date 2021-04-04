pragma solidity ^0.4.0;


//Creating Registration Smart contract

contract Registration {
    
    address public CDR; //Ethereum address of the CDR
    mapping(address => bool) public manufacturer; //a mapping that lists all authorized manufacturers
    mapping(address => bool) public distributor; //a mapping that lists all authorized distributors
    mapping(address => bool) public hospital; //a mapping for all authorized hospitals
    mapping(address => bool) public prescriber; //a mapping for authorized prescribers
    mapping(address => bool) public nurse; //a mapping for authorized nurses
    
    //Registration Events 
    
    event RegistrationSCDeployer (address indexed CDR); //An event to show the address of the registration SC deployer

    //Modifiers
    
    modifier onlyCDR() {
        require(CDR == msg.sender, "Only the CDR is eligible to run this function");
        _;
    }
    
    modifier onlyHospital {
        require(hospital [msg.sender], "Only the hospital is eligible to run this function");
        _;
    }
    
    //Creating the contract constructor

    constructor() public {
        
        CDR = msg.sender;
        emit RegistrationSCDeployer(CDR);

    }
    
    //Registration Functions
    
    function manufacturerRegistration (address user) public onlyCDR {
        
        manufacturer[user] = true;
        
    }
    
    function distributorRegistration (address user) public onlyCDR {
        
        distributor[user] = true;
    }
    
    function hospitalRegistration (address user) public onlyCDR {
        
        hospital[user] = true;
    }
    
    function prescriberRegistration (address user) public onlyHospital{
        
        prescriber[user] = true;
    }
    
    function nurseRegistration (address user) public onlyHospital{
        nurse[user] = true;
    }
    
}

//Creating the Supply Chain Smart contract

contract SupplyChain{
    
    //Declaring variables
    
    Registration public regcontract; //used to access variables and functions from the other contract
    string public controlleddrugname;
    string public lotbarcode; //Unique barcode is generated for each lot (ITF-14 type is assumed here)
    string public IPFShash; // A string variable that contains the hash of the uploadeed image
    enum  drugSchedule {TypeI, TypeII} //Note that schedules III, IV, V are not included
    drugSchedule  schedule; // the schedule variable can take either I or II as a value
    uint  public controlleddrugLots; //The amount of lots manufactured of the controlled drug
    uint  public controlleddrugQuantity; //The number of controlled drugs within each lot
    enum  controlleddruglotState  {NotReady, Manufactured, EnRoute, DeliveryEnded, Received, UnPackaged}
    controlleddruglotState public lotstate; //Refers to the state of the controlled drug container
    //enum controlleddrugstate {Prescribed, Administered, Disposed}
    //controlleddrugstate public drugstate; //refers to the state of the controlled drug after unboxing
    
    //Events 
    
    event SupplyChainSCDeployer (address indexed _address); //An event to show the address of the registration SC deployer
    event ControlledDrugManufactured (address indexed _manufacturer, string _barcode, string _controlleddrugname, string IPFShash, string _schedule, uint _controlleddrugLots, uint _controlleddrugQuantity);
    event StartDelivery (address indexed distributor); //The start of delivery is emitted by the distributor
    event EnRoute (address indexed distributor); //Event indicating that the controlled drug lot is being delivered
    event EndDelivery (address indexed distributor); // Event declaring the end of the delivery process
    event Reception (address indexed hospital); //Event confirming that the hospital has received the controlled drug lot
    
    //Creating Modifiers
    
    modifier onlyManufacturer{
        
        require(regcontract.manufacturer(msg.sender), "Only the manufacturer is allowed to execute this function");
        _;
    }
    
    modifier onlyDistributor{
    
        require(regcontract.distributor(msg.sender), "Only the distributor is allowed to execute this function");
        _;
    }
    
    modifier onlyHospital{
    
        require(regcontract.hospital(msg.sender), "Only the hospital is allowed to execute this function");
        _;
    }
    
    //Creating the contract constructor

    constructor(address registrationaddress) public {
        
        regcontract = Registration(registrationaddress); //links both contracts by setting the address of the registration contract
        regcontract.CDR();
        emit SupplyChainSCDeployer(regcontract.CDR()); //Should be changed to msg.sender if someone else will deploy the SC other than the CDR

    }
    

    
    //SupplyChain contract Functions
    
    
    function CreateControlledDrug(string _controlleddrugname, string _barcode, string _IPFShash, drugSchedule _schedule, uint _controlleddrugLots, uint _controlleddrugQuantity ) public onlyManufacturer returns(uint){
    require(lotstate == controlleddruglotState.NotReady, "can't run this function as the controlled drug lot has already been manufactured");
    lotstate == controlleddruglotState.Manufactured;
    lotbarcode = _barcode;
    controlleddrugname = _controlleddrugname;
    IPFShash = _IPFShash; //The hash can be converted into QR code and displayed on the DApp for easier access 
    controlleddrugLots = _controlleddrugLots;
    controlleddrugQuantity = _controlleddrugQuantity;
    
    if (_schedule == drugSchedule.TypeI){
        
        emit ControlledDrugManufactured(msg.sender, lotbarcode, controlleddrugname, IPFShash, "TypeI" , controlleddrugLots, controlleddrugQuantity );
    }
    else if (_schedule == drugSchedule.TypeII) {
        
        emit ControlledDrugManufactured(msg.sender,lotbarcode,  controlleddrugname, IPFShash, "TypeII" , controlleddrugLots, controlleddrugQuantity );
    }
    return  controlleddrugQuantity;
    }
    
    function startDelivey() public onlyDistributor{
        require(lotstate == controlleddruglotState.Manufactured, "Can't run this function as the controlled drug lot has already been delivered or not yet manufactured");
        lotstate = controlleddruglotState.EnRoute;
        emit StartDelivery(msg.sender);
    }
    
    function endDelivery() public onlyDistributor{
        require(lotstate == controlleddruglotState.EnRoute, "Can't run this function as it has already been received or not out for delivery ");
        lotstate = controlleddruglotState.DeliveryEnded;
        emit EndDelivery(msg.sender);
    }
    
    
    function LotReception() public onlyHospital{
        require(lotstate == controlleddruglotState.DeliveryEnded, "Can't run this function as it has already been received or still in en route");
        lotstate = controlleddruglotState.Received;
        emit Reception(msg.sender);
    }
    
    
    
    
    
}

//Creating the Consumption Smart contract

contract Consumption{
    
    //Declaring variables
    
    Registration regcontract2; 
    SupplyChain schain;
    
    string patientID;
    string patientName;
    uint patientAge;
    uint prescriptionDate;
    string endorsements; //The process of endorsing confirms the exact items that have been dispensed to the patient
    string prescriptionIPFShash; //A photo of the prescription is stored on thee IPFS
    string nurseName;
    uint administrationDate;
    uint availableAmount; 
    uint dispensedAmount;//The number of controlled drugs dispensed by nurses
    string sheetIPFShash; 
    uint disposedAmount; //The number of unwanted/unused controlled drugs that have been disposedAmount
    uint disposalDate;
    string disposalsheetIPFShash;
    
    enum controlleddrugstate {ReadyForDispensing, Prescribed, Administered, Disposed}
    controlleddrugstate public drugstate; //refers to the state of the controlled drug after unboxing
        
    //Events  
    
    event ConsumptionSCDeployer(address indexed _address); //shows the address of the SC deployer
    event DrugReady(address indexed _hospital);
    event DrugPrescribed (address indexed prescriber, string patientID, string patientName, uint patientAge, string endorsements, string prescriptionIPFShash);
    event DrugAdministered(address indexed _nurse, string nurseName, uint administrationDate, uint dispensedAmount, string sheetIPFShash);
    event DrugDisposed(address indexed nurse, string nurseName, uint disposalDate, string disposalsheetIPFShash);
    
    //Modifiers
    
    modifier onlyHospital{
    
        require(regcontract2.hospital(msg.sender), "Only the hospital is allowed to execute this function");
        _;
    }
    
    modifier onlyPrescriber{
    
        require(regcontract2.prescriber(msg.sender), "Only the prescriber is allowed to execute this function");
        _;
    }
 
     modifier onlyNurse{
    
        require(regcontract2.nurse(msg.sender), "Only the nurse is allowed to execute this function");
        _;
    }
       
    
    //Constructor 
    
    constructor(address registrationaddress, address supplyaddress) public {
        
        regcontract2 = Registration(registrationaddress);
        schain = SupplyChain(supplyaddress);
        //regcontract2.CDR()
        emit ConsumptionSCDeployer(regcontract2.CDR()); //Should be changed to msg.sender if someone else will deploy the SC other than the CDR
        
    }
    

    //Consumption contract Functions
    
    
    function DrugReadyForDispensing(uint _availableAmount) public onlyHospital{
        drugstate == controlleddrugstate.ReadyForDispensing;
        availableAmount = _availableAmount;
        emit DrugReady(msg.sender);
        
    }
    
    function DrugPrescription(string _patientID, string _patientName, uint _patientAge, string _endorsements, string _prescriptionIPFShash) public onlyPrescriber{
        
        require(drugstate == controlleddrugstate.ReadyForDispensing , "Can't prescribe controlled drug before it's ready");
        patientID = _patientID;
        patientName = _patientName;
        patientAge = _patientAge;
        endorsements = _endorsements;
        prescriptionIPFShash = _prescriptionIPFShash;
        drugstate = controlleddrugstate.Prescribed; 
        emit DrugPrescribed(msg.sender, patientID, patientName, patientAge, endorsements, prescriptionIPFShash);
    }
    
    function DrugAdministration(string _nurseName, uint _administrationDate, uint _dispensedAmount, string _sheetIPFShash) public onlyNurse{
        require(drugstate == controlleddrugstate.Prescribed, "Controlled drugs must be prescribed first before administration");
        require(dispensedAmount >= availableAmount , "The dispensed amount must be greater than or equal to the available amount");
        nurseName = _nurseName;
        administrationDate = _administrationDate;
        dispensedAmount = _dispensedAmount;
        sheetIPFShash = _sheetIPFShash; //Administration sheet is uploaded
        availableAmount -= dispensedAmount; //Decrease available amount as drugs are dispensed
        emit DrugAdministered(msg.sender, nurseName, administrationDate, dispensedAmount, sheetIPFShash);
    }
    
    function DrugDisposal(string _nurseName, uint _disposalDate, string _disposalsheetIPFShash) public onlyNurse{
        require(drugstate == controlleddrugstate.Administered, "Can't dispose drugs before they have been administired");
        availableAmount -= disposedAmount;
        nurseName = _nurseName;
        disposalDate = _disposalDate;
        disposalsheetIPFShash = _disposalsheetIPFShash;
        emit DrugDisposed(msg.sender, nurseName, disposalDate, disposalsheetIPFShash);

    }
    
        
}
