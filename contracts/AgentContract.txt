pragma solidity ^0.4.0;

contract AgentContract {
    string _url = "https://eth.sensors.com/api?token=a43fert"; //with unique sensor specific token
    uint _rate = 10;
    
    function get() public constant returns (address agent, string url, uint rate) {
        return (address(this), _url, _rate); //current agent address for reference while calling escrow
    }
	
	function getValue(string url) public constant returns (string returnValue) {
		// Logic to get sensor value from url
		
		if(keccak256(url) == keccak256(_url))
		    return "0x12";
		else
		    return "0x34";
	}
}
