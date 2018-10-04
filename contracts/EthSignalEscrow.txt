pragma solidity ^0.4.24;

import "./AgentContract.sol";

contract EthSignalEscrow{
    address public customer;
    address public agent;
    
    uint public cost;
	string public url;
	uint balance;
	uint private timestampTx; //timestamp of transaction
    
    bool public isCustomerIn;
    bool public isAgentIn;
    
    enum TransState {NOT_INITIATED, AWAITING_PAYMENT, AWAITING_DELIVERY, COMLETE, CANCELED} 
    TransState public currentState;
    
    modifier isCustomer() {require(msg.sender == customer);_; }
    modifier isCostMatched(){ require(msg.value == cost); _;}
    modifier nextState(TransState _nextState) {require(currentState == _nextState); _;}
        
	// Step 1: Customer gets URL (with unique sensor specific token) and rate from Agent contract
	
	// Step 2: Initialize escrew with all parameters
    constructor(address _customer, address _agent, string _url, uint _cost) public {
        customer = _customer;
        agent = _agent;
		url = _url;
        cost = _cost;
		timestampTx = now; //now is an alias for block.timestamp, not really "now"
    }
    
	//Step 3: Both customer and agent have to call escrow
    function initContract() isCostMatched nextState(TransState.NOT_INITIATED) payable public{
        if(msg.sender == customer){
            isCustomerIn = true;
        }
        if(msg.sender == agent){
            isAgentIn = true;
        }
        if(isCustomerIn && isAgentIn){
            currentState = TransState.AWAITING_PAYMENT;
			//makePayment();
        }
		else if (isCustomerIn && !isAgentIn && now > timestampTx + 30 days) {
            // Freeze 30 days before release to customer. The customer has to remember to call this method after freeze period.
            selfdestruct(customer);
        }
    }
    
	//Step 4: make payment
    function makePayment() isCostMatched isCustomer nextState(TransState.AWAITING_PAYMENT) payable public {		
		
		address(this).transfer(cost); // from customer to esrow account
	    agent.transfer(cost);		 // from escrow account to agent
	    
		currentState = TransState.AWAITING_DELIVERY;
		//return true;
		//return false;		
    }
	
	//STEP 5: get value from sensor and send to customer
    function confirmDelivery() isCustomer nextState(TransState.AWAITING_DELIVERY) payable public returns (string value) {
        //call url and get value from agent
		AgentContract ac = AgentContract(agent); // with agent deployed address
		string _value = "";
		_value = ac.getValue(url);
		
		// if successful, then
		if(_value != "") {
			currentState = TransState.COMLETE;
			
			return _value;
		}
		else {
			throw; //error
		}
    }
	
	// If transaction is canceled
	function cancel() public {
        if (msg.sender == customer){
            isCustomerIn = false;
			currentState = TransState.CANCELED;
        } else if (msg.sender == agent){
            isAgentIn = false;
			currentState = TransState.CANCELED;
        }
        // if both customer and agent would like to cancel, money is returned to customer 
        if (!isCustomerIn && !isAgentIn){
            selfdestruct(customer);
        }
    }
    
	// Incase 
    function kill() public constant {
        if (msg.sender == address(this)) {
            selfdestruct(customer);
			currentState = TransState.CANCELED;
        }
    }
}
