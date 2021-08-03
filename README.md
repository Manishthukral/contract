# contract
This is a lottery contract
pragma solidity ^0.8.0;

interface IERC20 {
    function transfer(address recipient, uint256 amount) external returns (bool);
    function transferFrom(
        address sender,
        address recipient,
        uint256 amount
    ) external returns (bool);
}



contract lottery
{
    address _owner;
    
    struct Userinfo
    {
        uint256 stackamount;
        uint256 rewardsearn;
        uint256 timeperiod;
    }
    
    uint256 poolid;
    mapping(uint256 => address) poolinfo;
    mapping(uint256 =>mapping(address => Userinfo)) userinformation;
    mapping(uint256 => uint256) poolparticipant;
    mapping(uint256 => uint256) dailylotteryrewardamount;
    mapping(uint256 => address) participantaddress;
    
    constructor()
    {
        _owner = msg.sender;
    }
    
    
    modifier onlyOwner() {
    require(isOwner(),"You are not authenticate to make this transfer");
    _;
     }
  
    function isOwner() internal view returns (bool) {
      return msg.sender == _owner;
     }
    
    function createpool(address _tokenaddress) public onlyOwner returns(bool)
    {
        poolid+=uint256(1);
        poolinfo[poolid]=_tokenaddress;
        return true;
    }
    
    function stakepooltoken(uint256 _poolid,uint256 amount) public returns(bool)
    {
       
       uint256 participant = poolparticipant[_poolid]+1;
       IERC20(poolinfo[_poolid]).transferFrom(msg.sender,address(this),amount);
       userinformation[_poolid][msg.sender].stackamount = amount;
       userinformation[_poolid][msg.sender].timeperiod = block.timestamp;
       poolparticipant[_poolid] = participant;
       participantaddress[participant]=msg.sender;
       return true;
    }
    
    function loot(uint256 _poolid) internal returns(uint256)
    {
        dailylotteryrewardamount[_poolid] = poolparticipant[_poolid]*(uint256(10));
        uint256 rand1 = uint(keccak256(abi.encodePacked(_poolid,dailylotteryrewardamount[_poolid],poolinfo[_poolid],poolparticipant[_poolid],block.timestamp))) % (poolparticipant[_poolid]);
        return rand1;
    }
    
    function winnerannouncing(uint256 _poolid) public
    {
        uint256 winningnumber = loot(_poolid);
        userinformation[_poolid][(participantaddress[winningnumber])].stackamount += dailylotteryrewardamount[_poolid];
        userinformation[_poolid][(participantaddress[winningnumber])].rewardsearn += dailylotteryrewardamount[_poolid];
    }
    
    function claming(uint256 _poolid) public
    {
        
    }
    
}
