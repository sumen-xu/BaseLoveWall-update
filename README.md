# BaseLoveWall-update
BaseLoveWall.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.34;

contract BaseLoveWall {
    address public immutable owner;
    uint256 public highlightFee = 0.0005 ether;
    uint256 public superLikeFee = 0.0001 ether;

    enum Mode { Love, Fun, Roast }

    struct Post {
        uint256 id;
        address author;
        string message;
        Mode mode;
        uint256 timestamp;
        uint256 likes;
        uint256 superLikes;
        bool isHighlighted;
    }

    Post[] public allPosts;

    mapping(uint256 => mapping(address => bool)) public hasLiked;
    mapping(uint256 => mapping(address => bool)) public hasSuperLiked;

    event PostCreated(uint256 indexed id, address author, string message, Mode mode, bool highlighted);
    event Liked(uint256 indexed postId, address liker);
    event SuperLiked(uint256 indexed postId, address liker);

    constructor() {
        owner = msg.sender;
    }

    modifier onlyOwner() {
        require(msg.sender == owner, "Only owner");
        _;
    }

    function postMessage(string memory _message, Mode _mode, bool _highlight) external payable {
        require(bytes(_message).length > 0 && bytes(_message).length <= 280, "Message must be 1-280 chars");

        bool highlighted = _highlight && msg.value == highlightFee;

        allPosts.push(Post({
            id: allPosts.length,
            author: msg.sender,
            message: _message,
            mode: _mode,
            timestamp: block.timestamp,
            likes: 0,
            superLikes: 0,
            isHighlighted: highlighted
        }));

        emit PostCreated(allPosts.length - 1, msg.sender, _message, _mode, highlighted);
    }

    function likePost(uint256 _postId) external {
        require(_postId < allPosts.length, "Post does not exist");
        require(!hasLiked[_postId][msg.sender], "Already liked");

        hasLiked[_postId][msg.sender] = true;
        allPosts[_postId].likes++;

        emit Liked(_postId, msg.sender);
    }

    function superLikePost(uint256 _postId) external payable {
        require(_postId < allPosts.length, "Post does not exist");
        require(!hasSuperLiked[_postId][msg.sender], "Already super liked");
        require(msg.value == superLikeFee, "Super Like costs exactly 0.0001 ETH");

        hasSuperLiked[_postId][msg.sender] = true;
        allPosts[_postId].superLikes++;

        emit SuperLiked(_postId, msg.sender);
    }

    function getLatestPosts(uint256 _count) external view returns (Post[] memory) {
        uint256 start = allPosts.length > _count ? allPosts.length - _count : 0;
        Post[] memory latest = new Post[](_count);
        for (uint i = 0; i < _count && start + i < allPosts.length; i++) {
            latest[i] = allPosts[start + i];
        }
        return latest;
    }

    function getTopPosts(uint256 _count) external view returns (Post[] memory) {
        Post[] memory top = new Post[](_count);
        uint256 start = allPosts.length > 50 ? allPosts.length - 50 : 0;
        for (uint i = 0; i < _count && start + i < allPosts.length; i++) {
            top[i] = allPosts[start + i];
        }
        return top;
    }

    function getTotalPosts() external view returns (uint256) {
        return allPosts.length;
    }

    function withdraw() external onlyOwner {
        (bool sent, ) = payable(owner).call{value: address(this).balance}("");
        require(sent, "Failed to send Ether");
    }

    function getContractInfo() external pure returns (string memory) {
        return "BaseLoveWall - Post Love, Fun or Roast messages on-chain and build the community!";
    }
}
