// SPDX-License-Identifier: MIT
pragma solidity 0.8.17;

import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/finance/PaymentSplitter.sol";
import "@openzeppelin/contracts/utils/cryptography/MerkleProof.sol";
import "@openzeppelin/contracts/utils/Strings.sol";
import "./ERC721A.sol";

contract NFTDemoERC721A is ERC721A, Ownable, PaymentSplitter {

    using Strings for uint;

    enum Step {
        Before,
        FreeMint,
        Between,
        WhitelistSale,
        PublicSale,
        Finished
    }

    Step public step;

    uint private constant MAX_SUPPLY = 101;
    uint private constant MAX_FREEMINT = 20;
    uint private constant MAX_WHITELIST = 30;
    uint private constant MAX_PUBLIC = 51;

    uint private constant PRICE_WHITELIST = 0.01 ether;
    uint private constant PRICE_PUBLIC = 0.02 ether;

    uint public saleStartTime = 1663616241;

    bytes32 public merkleRoot;

    string public baseURI;

    mapping(address => uint) amountNFTsperWalletFreemint;
    mapping(address => uint) amountNFTsperWalletWhitelist;
    mapping(address => uint) amountNFTsperWalletPublic;

    uint private constant MAX_PER_ADDRESS_DURING_FREEMINT = 2;
    uint private constant MAX_PER_ADDRESS_DURING_WHITELIST = 3;
    uint private constant MAX_PER_ADDRESS_DURING_PUBLIC = 5;

    uint private teamLength;

    address[] private _team = [
        0xb1EDbC29E9045Be3FFeFec2F0b2238EA8a03Ca38,
        0x77FEBfe7ECACb956b4FBd01d3c3Ae5D960219AEE,
        0x4ab7A2C230c878464C8C26a81F8F55f32AFE7122
    ];

    uint[] private _teamShares = [
        50,
        45,
        5
    ];

    constructor(bytes32 _merkleRoot, string memory _baseURI) ERC721A("DEMO NFT POKEMON 1er  GEN", "DNP") PaymentSplitter(_team, _teamShares) {
        merkleRoot = _merkleRoot;
        baseURI = _baseURI;
        teamLength = _team.length;
    }

    function freeMint(address _account, uint _quantity) external {
        require(getStep() == Step.FreeMint, "Vous devez attendre le debut du Freemint");
        require(amountNFTsperWalletFreemint[msg.sender] + _quantity <= MAX_PER_ADDRESS_DURING_FREEMINT, "Vous ne pouvez pas mint plus de 2 NFTs pendant le Freemint.");
        require(totalSupply() + _quantity <= MAX_FREEMINT, "Tout les Freemint sont mint");
        amountNFTsperWalletFreemint[msg.sender] += _quantity;
        _safeMint(_account, _quantity);
  
    }

    function whitelistMint(address _account, uint _quantity, bytes32[] calldata _proof) external payable {
        require(getStep() == Step.WhitelistSale, "Ce n est pas encore le moment de la Whitelist Sale vous ne pouvez pas mint");
        require(isWhitelisted(_account, _proof), "Vous n etes pas Whitelisted");
        require(amountNFTsperWalletWhitelist[msg.sender] + _quantity <= MAX_PER_ADDRESS_DURING_WHITELIST, "Vous ne pouvez pas mint plus de 3 NFT pendant la Whitelist Sale");
        require(totalSupply() + _quantity <= MAX_WHITELIST + MAX_FREEMINT, "Il ne reste plus assez de NFT pour cette phase");
        require(msg.value >= PRICE_WHITELIST * _quantity, "Pas assez de fonds");
        amountNFTsperWalletWhitelist[msg.sender] += _quantity;
        _safeMint(_account, _quantity);
    }

    function publicMint(address _account, uint _quantity) external payable {
        require(getStep() == Step.PublicSale, "Ce n est pas encore le Moment de la Public Sale vous ne pouvez pas mint");
        require(amountNFTsperWalletPublic[msg.sender] + _quantity <= MAX_PER_ADDRESS_DURING_PUBLIC, "Vous ne pouvez pas mint plus de 5 NFT pendant la Public Sale");
        require(totalSupply() + _quantity <= MAX_PUBLIC + MAX_WHITELIST + MAX_FREEMINT, "Il ne reste plus assez de NFT pour cette phase");
        require(msg.value >= PRICE_PUBLIC * _quantity, "Pas assez de fonds");
        amountNFTsperWalletPublic[msg.sender] += _quantity;
        _safeMint(_account, _quantity);
    }

    function getStep() public view returns(Step actualStep) {
        if(block.timestamp < saleStartTime) {
            return Step.Before;
        }
        if(block.timestamp >= saleStartTime
        && block.timestamp < saleStartTime + 12 hours){
            return Step.FreeMint;
        }
        if(block.timestamp >= saleStartTime + 12 hours
        && block.timestamp < saleStartTime + 24 hours) {
            return Step.Between;
        }
        if(block.timestamp >= saleStartTime + 24 hours
        && block.timestamp < saleStartTime + 36 hours) {
            return Step.WhitelistSale;
        }
        if(block.timestamp >= saleStartTime + 36 hours
        && block.timestamp < saleStartTime + 48 hours) {
            return Step.PublicSale;
        }
        if(block.timestamp >= saleStartTime + 48 hours
        && block.timestamp < saleStartTime + 60 hours) {
            return Step.Finished;
        }
    }

    function isWhitelisted(address _account, bytes32[] calldata _proof) internal view returns(bool) {
        return MerkleProof.verify(_proof, merkleRoot, keccak256(abi.encodePacked(_account)));
    }

    function tokenURI(uint _tokenId) public view virtual override(ERC721A)
     returns(string memory) {
        require(_exists(_tokenId), "Le NFT n est pas mint");

        return string(abi.encodePacked(baseURI, _tokenId.toString(), ".json"));
    }

    function setSaleStartTime(uint _saleStartTime) external onlyOwner {
        saleStartTime = _saleStartTime;
    }

    function setBaseURI(string memory _baseURI) external onlyOwner {
        baseURI = _baseURI;
    }

    function setMerkleRoot(bytes32 _merkleRoot) external onlyOwner {
        merkleRoot = _merkleRoot;
    }

    function releaseAll() external {
        for(uint i = 0 ; i < teamLength ; i++) {
            release(payable(payee(i)));
        }
    }

    receive() override external payable {
        revert("Seulement si tu mint");
    }
}
