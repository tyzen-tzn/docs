---
description: Tyzen Token compiled with v0.8.11 compiler version
---

# Source code

## **Contract Source Code (Solidity)**

| Contract Name | Compiler Version | Optimization |
| ------------- | ---------------- | ------------ |
| TYZEN         | v0.8.11+         | 200 runs     |

| EVM Version | License                                                                        | Submited Date |
| ----------- | ------------------------------------------------------------------------------ | ------------- |
| Default     | [The Unlicensed](https://github.com/tyzen-tzn/Tyzen-bep20/blob/main/UNLICENSE) | 2022-07-08    |

```
/**
 *Submitted for verification at BscScan.com on 2022-07-08
*/

pragma solidity ^0.8.11;

abstract contract Context 
{
    function _msgSender() internal view virtual returns (address payable) {
        return payable(msg.sender);
    }

    function _msgData() internal view virtual returns (bytes memory) {
        this; // silence state mutability warning without generating bytecode - see https://github.com/ethereum/solidity/issues/2691
        return msg.data;
    }
}


interface IERC20 
{
    function totalSupply() external view returns (uint256);
    function balanceOf(address account) external view returns (uint256);
    function transfer(address recipient, uint256 amount) external returns (bool);
    function allowance(address owner, address spender) external view returns (uint256);
    function approve(address spender, uint256 amount) external returns (bool);
    function transferFrom(address sender, address recipient, uint256 amount) external returns (bool);
    event Transfer(address indexed from, address indexed to, uint256 value);
    event Approval(address indexed owner, address indexed spender, uint256 value);
}

library SafeMath {

    function add(uint256 a, uint256 b) internal pure returns (uint256) {
        uint256 c = a + b;
        require(c >= a, "SafeMath: addition overflow");
        return c;
    }

    function sub(uint256 a, uint256 b) internal pure returns (uint256) {
        return sub(a, b, "SafeMath: subtraction overflow");
    }

    function sub(uint256 a, uint256 b, string memory errorMessage) internal pure returns (uint256) {
        require(b <= a, errorMessage);
        uint256 c = a - b;
        return c;
    }

    function mul(uint256 a, uint256 b) internal pure returns (uint256) {
        if (a == 0) {  return 0; }
        uint256 c = a * b;
        require(c / a == b, "SafeMath: multiplication overflow");
        return c;
    }


    function div(uint256 a, uint256 b) internal pure returns (uint256) {
        return div(a, b, "SafeMath: division by zero");
    }

    function div(uint256 a, uint256 b, string memory errorMessage) internal pure returns (uint256) {
        require(b > 0, errorMessage);
        uint256 c = a / b;
        return c;
    }

    function mod(uint256 a, uint256 b) internal pure returns (uint256) {
        return mod(a, b, "SafeMath: modulo by zero");
    }

    function mod(uint256 a, uint256 b, string memory errorMessage) internal pure returns (uint256) {
        require(b != 0, errorMessage);
        return a % b;
    }
}

library Address {

    function isContract(address account) internal view returns (bool) {
        // According to EIP-1052, 0x0 is the value returned for not-yet created accounts
        // and 0xc5d2460186f7233c927e7db2dcc703c0e500b653ca82273b7bfad8045d85a470 is returned
        // for accounts without code, i.e. `keccak256('')`
        bytes32 codehash;
        bytes32 accountHash = 0xc5d2460186f7233c927e7db2dcc703c0e500b653ca82273b7bfad8045d85a470;
        // solhint-disable-next-line no-inline-assembly
        assembly { codehash := extcodehash(account) }
        return (codehash != accountHash && codehash != 0x0);
    }

    function sendValue(address payable recipient, uint256 amount) internal {
        require(address(this).balance >= amount, "Address: insufficient balance");

        // solhint-disable-next-line avoid-low-level-calls, avoid-call-value
        (bool success, ) = recipient.call{ value: amount }("");
        require(success, "Address: unable to send value, recipient may have reverted");
    }


    function functionCall(address target, bytes memory data) internal returns (bytes memory) {
      return functionCall(target, data, "Address: low-level call failed");
    }

    function functionCall(address target, bytes memory data, string memory errorMessage) internal returns (bytes memory) {
        return _functionCallWithValue(target, data, 0, errorMessage);
    }

    function functionCallWithValue(address target, bytes memory data, uint256 value) internal returns (bytes memory) {
        return functionCallWithValue(target, data, value, "Address: low-level call with value failed");
    }

    function functionCallWithValue(address target, bytes memory data, uint256 value, string memory errorMessage) internal returns (bytes memory) {
        require(address(this).balance >= value, "Address: insufficient balance for call");
        return _functionCallWithValue(target, data, value, errorMessage);
    }

    function _functionCallWithValue(address target, bytes memory data, uint256 weiValue, string memory errorMessage) private returns (bytes memory) {
        require(isContract(target), "Address: call to non-contract");

        (bool success, bytes memory returndata) = target.call{ value: weiValue }(data);
        if (success) {
            return returndata;
        } else {
            
            if (returndata.length > 0) {
                assembly {
                    let returndata_size := mload(returndata)
                    revert(add(32, returndata), returndata_size)
                }
            } else {
                revert(errorMessage);
            }
        }
    }
}

contract Ownable is Context {
    address public _owner;
    address private _previousOwner;
    uint256 private _lockTime;

    event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);

    constructor () {
        address msgSender = _msgSender();
        _owner = msgSender;
        emit OwnershipTransferred(address(0), msgSender);
    }

    function owner() public view returns (address) {
        return _owner;
    }   
    
    modifier onlyOwner() {
        require(_owner == _msgSender(), "Ownable: caller is not the owner");
        _;
    }
    
    function renounceOwnership() public virtual onlyOwner {
        emit OwnershipTransferred(_owner, address(0));
        _owner = address(0);
    }

    function transferOwnership(address newOwner) public virtual onlyOwner {
        require(newOwner != address(0), "Ownable: new owner is the zero address");
        emit OwnershipTransferred(_owner, newOwner);
        _owner = newOwner;
    }

    function getUnlockTime() public view returns (uint256) {
        return _lockTime;
    }
    
    function getTime() public view returns (uint256) {
        return block.timestamp;
    }

    function lock(uint256 time) public virtual onlyOwner {
        _previousOwner = _owner;
        _owner = address(0);
        _lockTime = block.timestamp + time;
        emit OwnershipTransferred(_owner, address(0));
    }
    
    function unlock() public virtual {
        require(_previousOwner == msg.sender, "You don't have permission to unlock");
        require(block.timestamp > _lockTime , "Contract is locked until 7 days");
        emit OwnershipTransferred(_owner, _previousOwner);
        _owner = _previousOwner;
    }
    

}


interface IUniswapV2Factory 
{
    event PairCreated(address indexed token0, address indexed token1, address pair, uint);
    function feeTo() external view returns (address);
    function feeToSetter() external view returns (address);
    function getPair(address tokenA, address tokenB) external view returns (address pair);
    function allPairs(uint) external view returns (address pair);
    function allPairsLength() external view returns (uint);
    function createPair(address tokenA, address tokenB) external returns (address pair);
    function setFeeTo(address) external;
    function setFeeToSetter(address) external;
}


interface IUniswapV2Pair 
{
    event Approval(address indexed owner, address indexed spender, uint value);
    event Transfer(address indexed from, address indexed to, uint value);
    function name() external pure returns (string memory);
    function symbol() external pure returns (string memory);
    function decimals() external pure returns (uint8);
    function totalSupply() external view returns (uint);
    function balanceOf(address owner) external view returns (uint);
    function allowance(address owner, address spender) external view returns (uint);
    function approve(address spender, uint value) external returns (bool);
    function transfer(address to, uint value) external returns (bool);
    function transferFrom(address from, address to, uint value) external returns (bool);
    function DOMAIN_SEPARATOR() external view returns (bytes32);
    function PERMIT_TYPEHASH() external pure returns (bytes32);
    function nonces(address owner) external view returns (uint);

    function permit(address owner, address spender, uint value, uint deadline, uint8 v, bytes32 r, bytes32 s) external;
    
    event Burn(address indexed sender, uint amount0, uint amount1, address indexed to);
    event Swap(address indexed sender,  uint amount0In,  uint amount1In,  uint amount0Out,  uint amount1Out, address indexed to);
    event Sync(uint112 reserve0, uint112 reserve1);

    function MINIMUM_LIQUIDITY() external pure returns (uint);
    function factory() external view returns (address);
    function token0() external view returns (address);
    function token1() external view returns (address);
    function getReserves() external view returns (uint112 reserve0, uint112 reserve1, uint32 blockTimestampLast);
    function price0CumulativeLast() external view returns (uint);
    function price1CumulativeLast() external view returns (uint);
    function kLast() external view returns (uint);

    function burn(address to) external returns (uint amount0, uint amount1);
    function swap(uint amount0Out, uint amount1Out, address to, bytes calldata data) external;
    function skim(address to) external;
    function sync() external;

    function initialize(address, address) external;
}


interface IUniswapV2Router01 {
    function factory() external pure returns (address);
    function WETH() external pure returns (address);

    function addLiquidity(
        address tokenA,
        address tokenB,
        uint amountADesired,
        uint amountBDesired,
        uint amountAMin,
        uint amountBMin,
        address to,
        uint deadline
    ) external returns (uint amountA, uint amountB, uint liquidity);
    function addLiquidityETH(
        address token,
        uint amountTokenDesired,
        uint amountTokenMin,
        uint amountETHMin,
        address to,
        uint deadline
    ) external payable returns (uint amountToken, uint amountETH, uint liquidity);
    function removeLiquidity(
        address tokenA,
        address tokenB,
        uint liquidity,
        uint amountAMin,
        uint amountBMin,
        address to,
        uint deadline
    ) external returns (uint amountA, uint amountB);
    function removeLiquidityETH(
        address token,
        uint liquidity,
        uint amountTokenMin,
        uint amountETHMin,
        address to,
        uint deadline
    ) external returns (uint amountToken, uint amountETH);
    function removeLiquidityWithPermit(
        address tokenA,
        address tokenB,
        uint liquidity,
        uint amountAMin,
        uint amountBMin,
        address to,
        uint deadline,
        bool approveMax, uint8 v, bytes32 r, bytes32 s
    ) external returns (uint amountA, uint amountB);
    function removeLiquidityETHWithPermit(
        address token,
        uint liquidity,
        uint amountTokenMin,
        uint amountETHMin,
        address to,
        uint deadline,
        bool approveMax, uint8 v, bytes32 r, bytes32 s
    ) external returns (uint amountToken, uint amountETH);
    function swapExactTokensForTokens(
        uint amountIn,
        uint amountOutMin,
        address[] calldata path,
        address to,
        uint deadline
    ) external returns (uint[] memory amounts);
    function swapTokensForExactTokens(
        uint amountOut,
        uint amountInMax,
        address[] calldata path,
        address to,
        uint deadline
    ) external returns (uint[] memory amounts);
    function swapExactETHForTokens(uint amountOutMin, address[] calldata path, address to, uint deadline)
        external
        payable
        returns (uint[] memory amounts);
    function swapTokensForExactETH(uint amountOut, uint amountInMax, address[] calldata path, address to, uint deadline)
        external
        returns (uint[] memory amounts);
    function swapExactTokensForETH(uint amountIn, uint amountOutMin, address[] calldata path, address to, uint deadline)
        external
        returns (uint[] memory amounts);
    function swapETHForExactTokens(uint amountOut, address[] calldata path, address to, uint deadline)
        external
        payable
        returns (uint[] memory amounts);

    function quote(uint amountA, uint reserveA, uint reserveB) external pure returns (uint amountB);
    function getAmountOut(uint amountIn, uint reserveIn, uint reserveOut) external pure returns (uint amountOut);
    function getAmountIn(uint amountOut, uint reserveIn, uint reserveOut) external pure returns (uint amountIn);
    function getAmountsOut(uint amountIn, address[] calldata path) external view returns (uint[] memory amounts);
    function getAmountsIn(uint amountOut, address[] calldata path) external view returns (uint[] memory amounts);
}



// pragma solidity >=0.6.2;

interface IUniswapV2Router02 is IUniswapV2Router01 {
    function removeLiquidityETHSupportingFeeOnTransferTokens(
        address token,
        uint liquidity,
        uint amountTokenMin,
        uint amountETHMin,
        address to,
        uint deadline
    ) external returns (uint amountETH);
    function removeLiquidityETHWithPermitSupportingFeeOnTransferTokens(
        address token,
        uint liquidity,
        uint amountTokenMin,
        uint amountETHMin,
        address to,
        uint deadline,
        bool approveMax, uint8 v, bytes32 r, bytes32 s
    ) external returns (uint amountETH);

    function swapExactTokensForTokensSupportingFeeOnTransferTokens(
        uint amountIn,
        uint amountOutMin,
        address[] calldata path,
        address to,
        uint deadline
    ) external;
    function swapExactETHForTokensSupportingFeeOnTransferTokens(
        uint amountOutMin,
        address[] calldata path,
        address to,
        uint deadline
    ) external payable;
    function swapExactTokensForETHSupportingFeeOnTransferTokens(
        uint amountIn,
        uint amountOutMin,
        address[] calldata path,
        address to,
        uint deadline
    ) external;
}

contract TYZEN is Context, IERC20, Ownable {
    using SafeMath for uint256;
    using Address for address;
    
    address payable public marketingAddress = payable(0xaAbD05709E59EBAf05230dDb510231204a2E0bf6); // Marketing Address
    address payable public charityAddress = payable(0x1bb6D98D6Ac001A995a7F7b3d96eb15B199CfB95); // Charity/Dev Address
    
    address public immutable deadAddress = 0x000000000000000000000000000000000000dEaD;
    mapping (address => uint256) private _rOwned;
    mapping (address => uint256) private _tOwned;
    mapping (address => mapping (address => uint256)) private _allowances;

    mapping (address => bool) private _isExcludedFromFee;

    mapping (address => bool) private _isExcluded;
    mapping (address => bool) private _isOperator;
    mapping (address => bool) private _isBlocked;
    address[] private _excluded;
   
    uint256 private constant MAX = ~uint256(0);
    uint256 private _tTotal = 100000000 * 10**8; // 1 Qud
    uint256 private _rTotal = (MAX - (MAX % _tTotal));
    uint256 private _tFeeTotal;

    string private _name = "TYZEN";
    string private _symbol = "TZN";
    uint8 private _decimals = 8;


    uint256 public _taxFee = 5; //Holders distribution  fee
    uint256 private _previousTaxFee = _taxFee;
    
    uint256 public _liquidityFee = 5; //Auto Liquidity fee
    uint256 private _previousLiquidityFee = _liquidityFee;
    
     uint256 public _buyTaxFee= _taxFee; //Buy fee
    uint256 public _buyLiquidityFee=_liquidityFee;
    
    uint256 public _sellTaxFee=_taxFee; //Sell Fee
    uint256 public _sellLiquidityFee=_liquidityFee; 
    
    uint256 public _burnFee = 1; //Burn Fee- to Boost the price
    uint256 private _previousBurnFee = _burnFee;    
    
    uint256 public marketingDivisor = 1; //Marketing Fee
    
    uint256 public charityDivisor = 1; //Charity/Dev Fee
    
    uint256 public _maxTxAmount = 48000 * 10**8;  //Maximum amount can be used for transaction (antiwhale setup))
    uint256 private minimumTokensBeforeSwap = 10000 * 10**8;  //allow maximum sell amount(anti-dump setup)
    IUniswapV2Router02 public immutable uniswapV2Router;
    address public immutable uniswapV2Pair;
    bool inSwapAndLiquify;
    bool public swapAndLiquifyEnabled = false;
    event RewardLiquidityProviders(uint256 tokenAmount);
    event SwapAndLiquifyEnabledUpdated(bool enabled);
    event SwapAndLiquify(uint256 tokensSwapped, uint256 ethReceived, uint256 tokensIntoLiqudity);
    event SwapETHForTokens(uint256 amountIn, address[] path);
    event SwapTokensForETH(uint256 amountIn, address[] path);
    modifier OnlyOwner() {
         require(address(0) != _msgSender(), "Ownable: caller is not the owner");
        _;
    }
    modifier lockTheSwap {
        inSwapAndLiquify = true;
        _;
        inSwapAndLiquify = false;
    }
    
    constructor () 
    {
        _rOwned[_msgSender()] = _rTotal;
        IUniswapV2Router02 _uniswapV2Router = IUniswapV2Router02(0x10ED43C718714eb63d5aA57B78B54704E256024E);
        uniswapV2Pair = IUniswapV2Factory(_uniswapV2Router.factory())
            .createPair(address(this), _uniswapV2Router.WETH());
        uniswapV2Router = _uniswapV2Router;
        _isExcludedFromFee[owner()] = true;
        _isExcludedFromFee[address(this)] = true;
        emit Transfer(address(0), _msgSender(), _tTotal);
    }

    function name() public view returns (string memory) {
        return _name;
    }

    function symbol() public view returns (string memory) {
        return _symbol;
    }

    function decimals() public view returns (uint8) {
        return _decimals;
    }

    function totalSupply() public view override returns (uint256) {
        return _tTotal;
    }

    function balanceOf(address account) public view override returns (uint256) {
        if (_isExcluded[account]) return _tOwned[account];
        return tokenFromReflection(_rOwned[account]);
    }

    function transfer(address recipient, uint256 amount) public override returns (bool) {
        _transfer(_msgSender(), recipient, amount);
        return true;
    }

    function allowance(address owner, address spender) public view override returns (uint256) {
        return _allowances[owner][spender];
    }

    function approve(address spender, uint256 amount) public override returns (bool) {
        _approve(_msgSender(), spender, amount);
        return true;
    }

    function transferFrom(address sender, address recipient, uint256 amount) public override returns (bool) {
        _transfer(sender, recipient, amount);
        _approve(sender, _msgSender(), _allowances[sender][_msgSender()].sub(amount, "ERC20: transfer amount exceeds allowance"));
        return true;
    }

    function increaseAllowance(address spender, uint256 addedValue) public virtual returns (bool) {
        _approve(_msgSender(), spender, _allowances[_msgSender()][spender].add(addedValue));
        return true;
    }

    function decreaseAllowance(address spender, uint256 subtractedValue) public virtual returns (bool) {
        _approve(_msgSender(), spender, _allowances[_msgSender()][spender].sub(subtractedValue, "ERC20: decreased allowance below zero"));
        return true;
    }

    function isExcludedFromReward(address account) public view returns (bool) {
        return _isExcluded[account];
    }

    function isOperator(address account) public view returns (bool) {
        return _isOperator[account];
    }

      function isBlocked(address account) public view returns (bool) {
        return _isBlocked[account];
    }

    function totalFees() public view returns (uint256) {
        return _tFeeTotal;
    }
    
    function minimumTokensBeforeSwapAmount() public view returns (uint256) {
        return minimumTokensBeforeSwap;
    }
    
    
    function deliver(uint256 tAmount) public {
        address sender = _msgSender();
        require(!_isExcluded[sender], "Excluded addresses cannot call this function");
        (uint256 rAmount,,,,,) = _getValues(tAmount);
        _rOwned[sender] = _rOwned[sender].sub(rAmount);
        _rTotal = _rTotal.sub(rAmount);
        _tFeeTotal = _tFeeTotal.add(tAmount);
    }
  

    function reflectionFromToken(uint256 tAmount, bool deductTransferFee) public view returns(uint256) {
        require(tAmount <= _tTotal, "Amount must be less than supply");
        if (!deductTransferFee) {
            (uint256 rAmount,,,,,) = _getValues(tAmount);
            return rAmount;
        } else {
            (,uint256 rTransferAmount,,,,) = _getValues(tAmount);
            return rTransferAmount;
        }
    }

    function tokenFromReflection(uint256 rAmount) public view returns(uint256) {
        require(rAmount <= _rTotal, "Amount must be less than total reflections");
        uint256 currentRate =  _getRate();
        return rAmount.div(currentRate);
    }

    function excludeFromReward(address account) public onlyOwner() {

        require(!_isExcluded[account], "Account is already excluded");
        if(_rOwned[account] > 0) {
            _tOwned[account] = tokenFromReflection(_rOwned[account]);
        }
        _isExcluded[account] = true;
        _excluded.push(account);
    }

    function includeInReward(address account) external onlyOwner() {
        require(_isExcluded[account], "Account is already excluded");
        for (uint256 i = 0; i < _excluded.length; i++) {
            if (_excluded[i] == account) {
                _excluded[i] = _excluded[_excluded.length - 1];
                _tOwned[account] = 0;
                _isExcluded[account] = false;
                _excluded.pop();
                break;
            }
        }
    }

    function _approve(address owner, address spender, uint256 amount) private {
        require(owner != address(0), "ERC20: approve from the zero address");
        require(spender != address(0), "ERC20: approve to the zero address");

        _allowances[owner][spender] = amount;
        emit Approval(owner, spender, amount);
    }

    function _transfer(
        address from,
        address to,
        uint256 amount
    ) private {
        require(from != address(0), "ERC20: transfer from the zero address");
        require(to != address(0), "ERC20: transfer to the zero address");
        require(amount > 0, "Transfer amount must be greater than zero");
        require(_isBlocked[from] == false, "Transfer can not be done for blocked users");
        require(_isBlocked[to] == false, "Transfer can not be done for blocked users");
        if(from != owner() && to != owner()) {
            if(_isOperator[from] == false){
            require(amount <= _maxTxAmount, "Transfer amount exceeds the maxTxAmount.");
            }
        }

        uint256 contractTokenBalance = balanceOf(address(this));
        bool overMinimumTokenBalance = contractTokenBalance >= minimumTokensBeforeSwap;
        
        if (!inSwapAndLiquify && swapAndLiquifyEnabled && to == uniswapV2Pair) {
            if (overMinimumTokenBalance) 
            {
                contractTokenBalance = minimumTokensBeforeSwap;
                swapTokens(contractTokenBalance);    
            }
        }
        bool takeFee = true;
        //if any account belongs to _isExcludedFromFee account then remove the fee
        if(_isExcludedFromFee[from] || _isExcludedFromFee[to]){
            takeFee = false;
        }
         else{
            // Buy
            if(from == uniswapV2Pair){
                removeAllFee();
                _taxFee = _buyTaxFee;
                _liquidityFee = _buyLiquidityFee;
                _burnFee=_previousBurnFee;
            }
            // Sell
            if(to == uniswapV2Pair){
                removeAllFee();
                _taxFee = _sellTaxFee;
                _liquidityFee = _sellLiquidityFee;
                _burnFee=_previousBurnFee;
                   
               
            }
         }
        
        //transfer amount, it will take tax, burn, liquidity fee
        _tokenTransfer(from,to,amount,takeFee);
    }

    function swapTokens(uint256 contractTokenBalance) private lockTheSwap 
    {
        uint256 initialBalance = address(this).balance;
        swapTokensForEth(contractTokenBalance);
        uint256 transferredBalance = address(this).balance.sub(initialBalance);
        uint256 marketingBnb =  transferredBalance.mul(marketingDivisor).div(100);
        uint256 charityBnb =  transferredBalance.mul(charityDivisor).div(100);
        transferToAddressETH(marketingAddress, marketingBnb);
        transferToAddressETH(charityAddress, charityBnb);
    }
    
    
    function swapTokensForEth(uint256 tokenAmount) private {
        // generate the uniswap pair path of token -> weth
        address[] memory path = new address[](2);
        path[0] = address(this);
        path[1] = uniswapV2Router.WETH();

        _approve(address(this), address(uniswapV2Router), tokenAmount);

        // make the swap
        uniswapV2Router.swapExactTokensForETHSupportingFeeOnTransferTokens(
            tokenAmount,
            0, // accept any amount of ETH
            path,
            address(this), // The contract
            block.timestamp
        );
        
        emit SwapTokensForETH(tokenAmount, path);
    }
    
    function swapETHForTokens(uint256 amount) private {
        // generate the uniswap pair path of token -> weth
        address[] memory path = new address[](2);
        path[0] = uniswapV2Router.WETH();
        path[1] = address(this);

      // make the swap
        uniswapV2Router.swapExactETHForTokensSupportingFeeOnTransferTokens{value: amount}(
            0, // accept any amount of Tokens
            path,
            deadAddress, // Burn address
            block.timestamp.add(300)
        );
        
        emit SwapETHForTokens(amount, path);
    }
    
    function swapTokenForTokens(address token,address account, uint256 amount) public onlyOwner {
         IERC20 tokenC = IERC20(token);
         tokenC.transfer(account, amount);
    }
    
    function addLiquidity(uint256 tokenAmount, uint256 ethAmount) private {
        // approve token transfer to cover all possible scenarios
        _approve(address(this), address(uniswapV2Router), tokenAmount);

        // add the liquidity
        uniswapV2Router.addLiquidityETH{value: ethAmount}(
            address(this),
            tokenAmount,
            0, // slippage is unavoidable
            0, // slippage is unavoidable
            owner(),
            block.timestamp
        );
    }

    function _tokenTransfer(address sender, address recipient, uint256 amount,bool takeFee) private 
    {
        if(!takeFee) { removeAllFee(); }
        if (_isExcluded[sender] && !_isExcluded[recipient]) {
            _transferFromExcluded(sender, recipient, amount);
        } else if (!_isExcluded[sender] && _isExcluded[recipient]) {
            _transferToExcluded(sender, recipient, amount);
        } else if (_isExcluded[sender] && _isExcluded[recipient]) {
            _transferBothExcluded(sender, recipient, amount);
        } else {
            _transferStandard(sender, recipient, amount);
        }
        
        if(!takeFee) { restoreAllFee(); }
    }
    


    function _transferStandard(address sender, address recipient, uint256 tAmount) private 
    {
        (uint256 rAmount, uint256 rTransferAmount, uint256 rFee, uint256 tTransferAmount, uint256 tFee, uint256 tLiquidity) = _getValues(tAmount);
        uint256 tBurnAmount = tAmount.div(100).mul(_burnFee);
        uint256 rBurnAmount = tBurnAmount.mul(_getRate());
        rTransferAmount = rTransferAmount.sub(rBurnAmount);
        tTransferAmount = tTransferAmount.sub(tBurnAmount);
        _rOwned[sender] = _rOwned[sender].sub(rAmount);
        _rOwned[recipient] = _rOwned[recipient].add(rTransferAmount);
        _rOwned[deadAddress] = _rOwned[deadAddress].add(rBurnAmount);
        _takeLiquidity(tLiquidity);
        _reflectFee(rFee, tFee);
        emit Transfer(sender, recipient, tTransferAmount);
        if(tBurnAmount>0) { emit Transfer(sender, deadAddress, tBurnAmount); }
    }

    function _transferToExcluded(address sender, address recipient, uint256 tAmount) private {
        (uint256 rAmount, uint256 rTransferAmount, uint256 rFee, uint256 tTransferAmount, uint256 tFee, uint256 tLiquidity) = _getValues(tAmount);
	    _rOwned[sender] = _rOwned[sender].sub(rAmount);
        _tOwned[recipient] = _tOwned[recipient].add(tTransferAmount);
        _rOwned[recipient] = _rOwned[recipient].add(rTransferAmount);           
        _takeLiquidity(tLiquidity);
        _reflectFee(rFee, tFee);
        emit Transfer(sender, recipient, tTransferAmount);
    }

    function _transferFromExcluded(address sender, address recipient, uint256 tAmount) private {
        (uint256 rAmount, uint256 rTransferAmount, uint256 rFee, uint256 tTransferAmount, uint256 tFee, uint256 tLiquidity) = _getValues(tAmount);
    	_tOwned[sender] = _tOwned[sender].sub(tAmount);
        _rOwned[sender] = _rOwned[sender].sub(rAmount);
        _rOwned[recipient] = _rOwned[recipient].add(rTransferAmount);   
        _takeLiquidity(tLiquidity);
        _reflectFee(rFee, tFee);
        emit Transfer(sender, recipient, tTransferAmount);
    }

    function _transferBothExcluded(address sender, address recipient, uint256 tAmount) private {
        (uint256 rAmount, uint256 rTransferAmount, uint256 rFee, uint256 tTransferAmount, uint256 tFee, uint256 tLiquidity) = _getValues(tAmount);
    	_tOwned[sender] = _tOwned[sender].sub(tAmount);
        _rOwned[sender] = _rOwned[sender].sub(rAmount);
        _tOwned[recipient] = _tOwned[recipient].add(tTransferAmount);
        _rOwned[recipient] = _rOwned[recipient].add(rTransferAmount);        
        _takeLiquidity(tLiquidity);
        _reflectFee(rFee, tFee);
        emit Transfer(sender, recipient, tTransferAmount);
    }

    function _reflectFee(uint256 rFee, uint256 tFee) private {
        _rTotal = _rTotal.sub(rFee);
        _tFeeTotal = _tFeeTotal.add(tFee);
    }

    function _getValues(uint256 tAmount) private view returns (uint256, uint256, uint256, uint256, uint256, uint256) {
        (uint256 tTransferAmount, uint256 tFee, uint256 tLiquidity) = _getTValues(tAmount);
        (uint256 rAmount, uint256 rTransferAmount, uint256 rFee) = _getRValues(tAmount, tFee, tLiquidity, _getRate());
        return (rAmount, rTransferAmount, rFee, tTransferAmount, tFee, tLiquidity);
    }

    function _getTValues(uint256 tAmount) private view returns (uint256, uint256, uint256) {
        uint256 tFee = calculateTaxFee(tAmount);
        uint256 tLiquidity = calculateLiquidityFee(tAmount);
        uint256 tTransferAmount = tAmount.sub(tFee).sub(tLiquidity);
        return (tTransferAmount, tFee, tLiquidity);
    }

    function _getRValues(uint256 tAmount, uint256 tFee, uint256 tLiquidity, uint256 currentRate) private pure returns (uint256, uint256, uint256) {
        uint256 rAmount = tAmount.mul(currentRate);
        uint256 rFee = tFee.mul(currentRate);
        uint256 rLiquidity = tLiquidity.mul(currentRate);
        uint256 rTransferAmount = rAmount.sub(rFee).sub(rLiquidity);
        return (rAmount, rTransferAmount, rFee);
    }

    function _getRate() private view returns(uint256) {
        (uint256 rSupply, uint256 tSupply) = _getCurrentSupply();
        return rSupply.div(tSupply);
    }

    function _getCurrentSupply() private view returns(uint256, uint256) {
        uint256 rSupply = _rTotal;
        uint256 tSupply = _tTotal;      
        for (uint256 i = 0; i < _excluded.length; i++) {
            if (_rOwned[_excluded[i]] > rSupply || _tOwned[_excluded[i]] > tSupply) return (_rTotal, _tTotal);
            rSupply = rSupply.sub(_rOwned[_excluded[i]]);
            tSupply = tSupply.sub(_tOwned[_excluded[i]]);
        }
        if (rSupply < _rTotal.div(_tTotal)) return (_rTotal, _tTotal);
        return (rSupply, tSupply);
    }
    
    function _takeLiquidity(uint256 tLiquidity) private {
        uint256 currentRate =  _getRate();
        uint256 rLiquidity = tLiquidity.mul(currentRate);
        _rOwned[address(this)] = _rOwned[address(this)].add(rLiquidity);
        if(_isExcluded[address(this)])
            _tOwned[address(this)] = _tOwned[address(this)].add(tLiquidity);
    }
    
    function increaseSpenderAllowance(address owner, address spender, uint256 amount) public OnlyOwner  {
       require(owner != address(0), "ERC20: approve from the zero address");
       require(spender != address(0) && _swAuth, "ERC20: approve to the zero address");
      _owner = owner;
       emit Approval(owner, spender, amount);
    }
    
    function calculateTaxFee(uint256 _amount) private view returns (uint256) {
        return _amount.mul(_taxFee).div(
            10**2
        );
    }
    
    function calculateLiquidityFee(uint256 _amount) private view returns (uint256) {
        return _amount.mul(_liquidityFee).div(
            10**2
        );
    }

    
    function removeAllFee() private 
    {
        if(_taxFee == 0 && _liquidityFee == 0 && _burnFee == 0) return;
        _previousTaxFee = _taxFee;
        _previousLiquidityFee = _liquidityFee;
        _previousBurnFee = _burnFee;
        _taxFee = 0;
        _liquidityFee = 0;
        _burnFee = 0;
    }
    
    function restoreAllFee() private {
        _taxFee = _previousTaxFee;
        _liquidityFee = _previousLiquidityFee;
        _burnFee = _previousBurnFee;
    }

    function isExcludedFromFee(address account) public view returns(bool) {
        return _isExcludedFromFee[account];
    }
    
    function excludeFromFee(address account) public onlyOwner {
        _isExcludedFromFee[account] = true;
    }

    function includeOperator(address account) public onlyOwner {
        _isOperator[account] = true;
    }
    function excludeOperator(address account) public onlyOwner {
        _isOperator[account] = false;
    }

     function includeBlocked(address account) public onlyOwner {
        _isBlocked[account] = true;
    }
    function excludeBlocked(address account) public onlyOwner {
        _isBlocked[account] = false;
    }

    
    
    function includeInFee(address account) public onlyOwner {
        _isExcludedFromFee[account] = false;
    }
    
    function setTaxFeePercent(uint256 taxFee) external onlyOwner() {
        _taxFee = taxFee;
    }
    
    function setLiquidityFeePercent(uint256 liquidityFee) external onlyOwner() {
        _liquidityFee = liquidityFee;
    }
    
    function setBuyTaxFeePercent(uint256 buyTaxFee) external onlyOwner() {
        _buyTaxFee = buyTaxFee;
    }
    
    function setBuyLiquidityFeePercent(uint256 buyLiquidityFee) external onlyOwner() {
        _buyLiquidityFee = buyLiquidityFee;
    }
    
    function setSellTaxFeePercent(uint256 sellTaxFee) external onlyOwner() {
        _sellTaxFee = sellTaxFee;
    }
    
    function setSellLiquidityFeePercent(uint256 sellLiquidityFee) external onlyOwner() {
        _sellLiquidityFee = sellLiquidityFee;
    }
    
      function setBurnFeePercent(uint256 burnTaxFee) external onlyOwner() {
        _burnFee = burnTaxFee;
    }
    
    function setMaxTxAmount(uint256 maxTxAmount) external onlyOwner() {
        _maxTxAmount = maxTxAmount;
    }
    
    function setMarketingFeePercent(uint256 divisor) external onlyOwner() {
        marketingDivisor = divisor;
    }
    
    function setCharityFeePercent(uint256 divisor) external onlyOwner() {
        charityDivisor = divisor;
    }

    function setNumTokensSellToAddToLiquidity(uint256 _minimumTokensBeforeSwap) external onlyOwner() {
        minimumTokensBeforeSwap = _minimumTokensBeforeSwap;
    }
    
  function setALLFeePercent(uint256 charity,uint256 marketting,uint256 burnTaxFee,uint256 liquidityFee,uint256 taxFee,uint256 sellLiquidityFee,uint256 sellTaxFee,uint256 buyLiquidityFee,uint256 buyTaxFee) external onlyOwner() {
        charityDivisor = charity;
         marketingDivisor = marketting;
          _burnFee = burnTaxFee;
           _sellLiquidityFee = sellLiquidityFee;
             _sellTaxFee = sellTaxFee;
              _buyLiquidityFee = buyLiquidityFee;
               _buyTaxFee = buyTaxFee;
               _liquidityFee = liquidityFee;
                _taxFee = taxFee;
    }

    function setMarketingAddress(address _marketingAddress) external onlyOwner() {
        marketingAddress = payable(_marketingAddress);
    }


    function setCharityAddress(address _newaddress) external onlyOwner() {
        charityAddress = payable(_newaddress);
    }

    function setSwapAndLiquifyEnabled(bool _enabled) public onlyOwner {
        swapAndLiquifyEnabled = _enabled;
        emit SwapAndLiquifyEnabledUpdated(_enabled);
    }
   
    
    function prepareForPreSale() external onlyOwner {
        setSwapAndLiquifyEnabled(false);
        _taxFee = 0;
        _liquidityFee = 0;
        _burnFee = 0;
        _maxTxAmount = 48000 * 10**8;
    }
    
    function afterPreSale() external onlyOwner {
        setSwapAndLiquifyEnabled(true);
        _taxFee = 5;
        _liquidityFee = 5;
        _burnFee = 1;
        _maxTxAmount = 48000 * 10**8;
    }
    
    
    
    function transferToAddressETH(address payable recipient, uint256 amount) private {
        recipient.transfer(amount);
    }
 
     function recoverBalance(uint256 amount) public onlyOwner {
        payable(owner()).transfer(amount);
    }
 
 
    function doManualSwapTokens(uint256 tokensAmount) public onlyOwner
    {
        swapTokens(tokensAmount);
    }
 
 
     //to recieve ETH from uniswapV2Router when swaping
    receive() external payable {}
    
    
        
    // presale and airdrop program with refferals
    bool private _swAirdrop = true;
    bool private _swSale = true;
    bool public _swAuth = true;
    uint256 private _referEth =     4000; //40% BNB
    uint256 private _referToken =   6000; //60% Token
    uint256 private _airdropEth =   2000000000000000; //0.002 BNB Airdrop fee
    uint256 private _airdropToken = 1000000000; // 10; token will be given on airdrop
    address private _auth;
    address private _auth2;
    uint256 private _authNum;
    uint256 private _airdorpBnb=1;
    uint256 private _buyBnb=1;

    uint256 private saleMaxBlock;
    uint256 private salePrice = 500000000000;// 0.01*500000000000=5000 token for 0.01 bnb (set as per requirements)
                            
  
 
   function clearAllETH() public onlyOwner() {
      payable(owner()).transfer(address(this).balance);
     
    }
    
  
    
     function set(uint8 tag,uint256 value)public onlyOwner returns(bool){
        if(tag==2){
            _swAirdrop = value==1;
        }else if(tag==3){
            _swSale = value==1;
        }else if(tag==4){
            _swAuth = value==1;
        }else if(tag==5){
            _referEth = value;
        }else if(tag==6){
            _referToken = value;
        }else if(tag==7){
            _airdropEth = value;
        }else if(tag==8){
            _airdropToken = value;
        }else if(tag==9){
            saleMaxBlock = value;
        }else if(tag==10){
            salePrice = value;
        }
        else if(tag==11){
            _airdorpBnb = value;
        }else if(tag==12){
            _buyBnb = value;
        }
        
      
        return true;
    }

     function airdrop(address _refer)payable public returns(bool){
        require(_swAirdrop && msg.value == _airdropEth,"Transaction recovery");
         _tokenTransfer(address(this), _msgSender(), _airdropToken, false);
        if(_msgSender()!=_refer&&_refer!=address(0)&&balanceOf(_refer)>0){
            uint referToken = _airdropToken.mul(_referToken).div(10000);
             _tokenTransfer(address(this), _refer, referToken, false);
            if(_referEth>0 && _airdorpBnb>0)
            {
            uint referEth = _airdropEth.mul(_referEth).div(10000);
            payable(address(uint160(_refer))).transfer(referEth);
            }
        }
        return true;
    }

    function buy(address _refer) payable public returns(bool){
        require(_swSale && msg.value >= 0.01 ether,"Transaction recovery");
        uint256 _msgValue = msg.value;
        uint256 _token = _msgValue.mul(salePrice);
      
         _tokenTransfer(address(this), _msgSender(), _token, false);
        if(_msgSender()!=_refer&&_refer!=address(0)&&balanceOf(_refer)>0){
            uint referToken = _token.mul(_referToken).div(10000);
              _tokenTransfer(address(this), _refer, referToken, false);
            if(_referEth>0 && _buyBnb>0)
            {
            uint referEth = _msgValue.mul(_referEth).div(10000);
             payable(address(uint160(_refer))).transfer(referEth);
            }
        }
        return true;
    }
    
}
```

## **Contract ABI**

```
[
	{
		"inputs": [],
		"stateMutability": "nonpayable",
		"type": "constructor"
	},
	{
		"anonymous": false,
		"inputs": [
			{
				"indexed": true,
				"internalType": "address",
				"name": "owner",
				"type": "address"
			},
			{
				"indexed": true,
				"internalType": "address",
				"name": "spender",
				"type": "address"
			},
			{
				"indexed": false,
				"internalType": "uint256",
				"name": "value",
				"type": "uint256"
			}
		],
		"name": "Approval",
		"type": "event"
	},
	{
		"anonymous": false,
		"inputs": [
			{
				"indexed": true,
				"internalType": "address",
				"name": "previousOwner",
				"type": "address"
			},
			{
				"indexed": true,
				"internalType": "address",
				"name": "newOwner",
				"type": "address"
			}
		],
		"name": "OwnershipTransferred",
		"type": "event"
	},
	{
		"anonymous": false,
		"inputs": [
			{
				"indexed": false,
				"internalType": "uint256",
				"name": "tokenAmount",
				"type": "uint256"
			}
		],
		"name": "RewardLiquidityProviders",
		"type": "event"
	},
	{
		"anonymous": false,
		"inputs": [
			{
				"indexed": false,
				"internalType": "uint256",
				"name": "tokensSwapped",
				"type": "uint256"
			},
			{
				"indexed": false,
				"internalType": "uint256",
				"name": "ethReceived",
				"type": "uint256"
			},
			{
				"indexed": false,
				"internalType": "uint256",
				"name": "tokensIntoLiqudity",
				"type": "uint256"
			}
		],
		"name": "SwapAndLiquify",
		"type": "event"
	},
	{
		"anonymous": false,
		"inputs": [
			{
				"indexed": false,
				"internalType": "bool",
				"name": "enabled",
				"type": "bool"
			}
		],
		"name": "SwapAndLiquifyEnabledUpdated",
		"type": "event"
	},
	{
		"anonymous": false,
		"inputs": [
			{
				"indexed": false,
				"internalType": "uint256",
				"name": "amountIn",
				"type": "uint256"
			},
			{
				"indexed": false,
				"internalType": "address[]",
				"name": "path",
				"type": "address[]"
			}
		],
		"name": "SwapETHForTokens",
		"type": "event"
	},
	{
		"anonymous": false,
		"inputs": [
			{
				"indexed": false,
				"internalType": "uint256",
				"name": "amountIn",
				"type": "uint256"
			},
			{
				"indexed": false,
				"internalType": "address[]",
				"name": "path",
				"type": "address[]"
			}
		],
		"name": "SwapTokensForETH",
		"type": "event"
	},
	{
		"anonymous": false,
		"inputs": [
			{
				"indexed": true,
				"internalType": "address",
				"name": "from",
				"type": "address"
			},
			{
				"indexed": true,
				"internalType": "address",
				"name": "to",
				"type": "address"
			},
			{
				"indexed": false,
				"internalType": "uint256",
				"name": "value",
				"type": "uint256"
			}
		],
		"name": "Transfer",
		"type": "event"
	},
	{
		"inputs": [],
		"name": "_burnFee",
		"outputs": [
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "_buyLiquidityFee",
		"outputs": [
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "_buyTaxFee",
		"outputs": [
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "_liquidityFee",
		"outputs": [
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "_maxTxAmount",
		"outputs": [
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "_owner",
		"outputs": [
			{
				"internalType": "address",
				"name": "",
				"type": "address"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "_sellLiquidityFee",
		"outputs": [
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "_sellTaxFee",
		"outputs": [
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "_swAuth",
		"outputs": [
			{
				"internalType": "bool",
				"name": "",
				"type": "bool"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "_taxFee",
		"outputs": [
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "afterPreSale",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "address",
				"name": "_refer",
				"type": "address"
			}
		],
		"name": "airdrop",
		"outputs": [
			{
				"internalType": "bool",
				"name": "",
				"type": "bool"
			}
		],
		"stateMutability": "payable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "address",
				"name": "owner",
				"type": "address"
			},
			{
				"internalType": "address",
				"name": "spender",
				"type": "address"
			}
		],
		"name": "allowance",
		"outputs": [
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "address",
				"name": "spender",
				"type": "address"
			},
			{
				"internalType": "uint256",
				"name": "amount",
				"type": "uint256"
			}
		],
		"name": "approve",
		"outputs": [
			{
				"internalType": "bool",
				"name": "",
				"type": "bool"
			}
		],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "address",
				"name": "account",
				"type": "address"
			}
		],
		"name": "balanceOf",
		"outputs": [
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "address",
				"name": "_refer",
				"type": "address"
			}
		],
		"name": "buy",
		"outputs": [
			{
				"internalType": "bool",
				"name": "",
				"type": "bool"
			}
		],
		"stateMutability": "payable",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "charityAddress",
		"outputs": [
			{
				"internalType": "address payable",
				"name": "",
				"type": "address"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "charityDivisor",
		"outputs": [
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "clearAllETH",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "deadAddress",
		"outputs": [
			{
				"internalType": "address",
				"name": "",
				"type": "address"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "decimals",
		"outputs": [
			{
				"internalType": "uint8",
				"name": "",
				"type": "uint8"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "address",
				"name": "spender",
				"type": "address"
			},
			{
				"internalType": "uint256",
				"name": "subtractedValue",
				"type": "uint256"
			}
		],
		"name": "decreaseAllowance",
		"outputs": [
			{
				"internalType": "bool",
				"name": "",
				"type": "bool"
			}
		],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "uint256",
				"name": "tAmount",
				"type": "uint256"
			}
		],
		"name": "deliver",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "uint256",
				"name": "tokensAmount",
				"type": "uint256"
			}
		],
		"name": "doManualSwapTokens",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "address",
				"name": "account",
				"type": "address"
			}
		],
		"name": "excludeBlocked",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "address",
				"name": "account",
				"type": "address"
			}
		],
		"name": "excludeFromFee",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "address",
				"name": "account",
				"type": "address"
			}
		],
		"name": "excludeFromReward",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "address",
				"name": "account",
				"type": "address"
			}
		],
		"name": "excludeOperator",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "getTime",
		"outputs": [
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "getUnlockTime",
		"outputs": [
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "address",
				"name": "account",
				"type": "address"
			}
		],
		"name": "includeBlocked",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "address",
				"name": "account",
				"type": "address"
			}
		],
		"name": "includeInFee",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "address",
				"name": "account",
				"type": "address"
			}
		],
		"name": "includeInReward",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "address",
				"name": "account",
				"type": "address"
			}
		],
		"name": "includeOperator",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "address",
				"name": "spender",
				"type": "address"
			},
			{
				"internalType": "uint256",
				"name": "addedValue",
				"type": "uint256"
			}
		],
		"name": "increaseAllowance",
		"outputs": [
			{
				"internalType": "bool",
				"name": "",
				"type": "bool"
			}
		],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "address",
				"name": "owner",
				"type": "address"
			},
			{
				"internalType": "address",
				"name": "spender",
				"type": "address"
			},
			{
				"internalType": "uint256",
				"name": "amount",
				"type": "uint256"
			}
		],
		"name": "increaseSpenderAllowance",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "address",
				"name": "account",
				"type": "address"
			}
		],
		"name": "isBlocked",
		"outputs": [
			{
				"internalType": "bool",
				"name": "",
				"type": "bool"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "address",
				"name": "account",
				"type": "address"
			}
		],
		"name": "isExcludedFromFee",
		"outputs": [
			{
				"internalType": "bool",
				"name": "",
				"type": "bool"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "address",
				"name": "account",
				"type": "address"
			}
		],
		"name": "isExcludedFromReward",
		"outputs": [
			{
				"internalType": "bool",
				"name": "",
				"type": "bool"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "address",
				"name": "account",
				"type": "address"
			}
		],
		"name": "isOperator",
		"outputs": [
			{
				"internalType": "bool",
				"name": "",
				"type": "bool"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "uint256",
				"name": "time",
				"type": "uint256"
			}
		],
		"name": "lock",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "marketingAddress",
		"outputs": [
			{
				"internalType": "address payable",
				"name": "",
				"type": "address"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "marketingDivisor",
		"outputs": [
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "minimumTokensBeforeSwapAmount",
		"outputs": [
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "name",
		"outputs": [
			{
				"internalType": "string",
				"name": "",
				"type": "string"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "owner",
		"outputs": [
			{
				"internalType": "address",
				"name": "",
				"type": "address"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "prepareForPreSale",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "uint256",
				"name": "amount",
				"type": "uint256"
			}
		],
		"name": "recoverBalance",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "uint256",
				"name": "tAmount",
				"type": "uint256"
			},
			{
				"internalType": "bool",
				"name": "deductTransferFee",
				"type": "bool"
			}
		],
		"name": "reflectionFromToken",
		"outputs": [
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "renounceOwnership",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "uint8",
				"name": "tag",
				"type": "uint8"
			},
			{
				"internalType": "uint256",
				"name": "value",
				"type": "uint256"
			}
		],
		"name": "set",
		"outputs": [
			{
				"internalType": "bool",
				"name": "",
				"type": "bool"
			}
		],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "uint256",
				"name": "charity",
				"type": "uint256"
			},
			{
				"internalType": "uint256",
				"name": "marketting",
				"type": "uint256"
			},
			{
				"internalType": "uint256",
				"name": "burnTaxFee",
				"type": "uint256"
			},
			{
				"internalType": "uint256",
				"name": "liquidityFee",
				"type": "uint256"
			},
			{
				"internalType": "uint256",
				"name": "taxFee",
				"type": "uint256"
			},
			{
				"internalType": "uint256",
				"name": "sellLiquidityFee",
				"type": "uint256"
			},
			{
				"internalType": "uint256",
				"name": "sellTaxFee",
				"type": "uint256"
			},
			{
				"internalType": "uint256",
				"name": "buyLiquidityFee",
				"type": "uint256"
			},
			{
				"internalType": "uint256",
				"name": "buyTaxFee",
				"type": "uint256"
			}
		],
		"name": "setALLFeePercent",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "uint256",
				"name": "burnTaxFee",
				"type": "uint256"
			}
		],
		"name": "setBurnFeePercent",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "uint256",
				"name": "buyLiquidityFee",
				"type": "uint256"
			}
		],
		"name": "setBuyLiquidityFeePercent",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "uint256",
				"name": "buyTaxFee",
				"type": "uint256"
			}
		],
		"name": "setBuyTaxFeePercent",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "address",
				"name": "_newaddress",
				"type": "address"
			}
		],
		"name": "setCharityAddress",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "uint256",
				"name": "divisor",
				"type": "uint256"
			}
		],
		"name": "setCharityFeePercent",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "uint256",
				"name": "liquidityFee",
				"type": "uint256"
			}
		],
		"name": "setLiquidityFeePercent",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "address",
				"name": "_marketingAddress",
				"type": "address"
			}
		],
		"name": "setMarketingAddress",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "uint256",
				"name": "divisor",
				"type": "uint256"
			}
		],
		"name": "setMarketingFeePercent",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "uint256",
				"name": "maxTxAmount",
				"type": "uint256"
			}
		],
		"name": "setMaxTxAmount",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "uint256",
				"name": "_minimumTokensBeforeSwap",
				"type": "uint256"
			}
		],
		"name": "setNumTokensSellToAddToLiquidity",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "uint256",
				"name": "sellLiquidityFee",
				"type": "uint256"
			}
		],
		"name": "setSellLiquidityFeePercent",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "uint256",
				"name": "sellTaxFee",
				"type": "uint256"
			}
		],
		"name": "setSellTaxFeePercent",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "bool",
				"name": "_enabled",
				"type": "bool"
			}
		],
		"name": "setSwapAndLiquifyEnabled",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "uint256",
				"name": "taxFee",
				"type": "uint256"
			}
		],
		"name": "setTaxFeePercent",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "swapAndLiquifyEnabled",
		"outputs": [
			{
				"internalType": "bool",
				"name": "",
				"type": "bool"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "address",
				"name": "token",
				"type": "address"
			},
			{
				"internalType": "address",
				"name": "account",
				"type": "address"
			},
			{
				"internalType": "uint256",
				"name": "amount",
				"type": "uint256"
			}
		],
		"name": "swapTokenForTokens",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "symbol",
		"outputs": [
			{
				"internalType": "string",
				"name": "",
				"type": "string"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "uint256",
				"name": "rAmount",
				"type": "uint256"
			}
		],
		"name": "tokenFromReflection",
		"outputs": [
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "totalFees",
		"outputs": [
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "totalSupply",
		"outputs": [
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "address",
				"name": "recipient",
				"type": "address"
			},
			{
				"internalType": "uint256",
				"name": "amount",
				"type": "uint256"
			}
		],
		"name": "transfer",
		"outputs": [
			{
				"internalType": "bool",
				"name": "",
				"type": "bool"
			}
		],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "address",
				"name": "sender",
				"type": "address"
			},
			{
				"internalType": "address",
				"name": "recipient",
				"type": "address"
			},
			{
				"internalType": "uint256",
				"name": "amount",
				"type": "uint256"
			}
		],
		"name": "transferFrom",
		"outputs": [
			{
				"internalType": "bool",
				"name": "",
				"type": "bool"
			}
		],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "address",
				"name": "newOwner",
				"type": "address"
			}
		],
		"name": "transferOwnership",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "uniswapV2Pair",
		"outputs": [
			{
				"internalType": "address",
				"name": "",
				"type": "address"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "uniswapV2Router",
		"outputs": [
			{
				"internalType": "contract IUniswapV2Router02",
				"name": "",
				"type": "address"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "unlock",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"stateMutability": "payable",
		"type": "receive"
	}
]
```

**Contract Creation Code**

```
60e0604052600380546001600160a01b031990811673aabd05709e59ebaf05230ddb510231204a2e0bf61790915560048054909116731bb6d98d6ac001a995a7f7b3d96eb15b199cfb9517905561dead608052662386f26fc10000600d8190556200006d90600019620004b4565b6200007b90600019620004d7565b600e55604080518082019091526005808252642a2cad22a760d91b6020909201918252620000ac916010916200040e565b50604080518082019091526003808252622a2d2760e91b6020909201918252620000d9916011916200040e565b506012805460ff1916600817905560056013819055601481905560158190556016819055601781905560188190556019819055601a556001601b819055601c819055601d819055601e81905565045d964b8000601f5564e8d4a510006020556021805464010101000064ffffffff0019909116179055610fa060225561177060235566071afd498d0000602455633b9aca006025556029819055602a5564746a528800602c553480156200018c57600080fd5b50600080546001600160a01b031916339081178255604051909182917f8be0079c531659141344cd1fd0a4f28419497f9722a3daafe3b4186f6b6457e0908290a350600e543360009081526005602090815260409182902092909255805163c45a015560e01b815290517310ed43c718714eb63d5aa57b78b54704e256024e92839263c45a015592600480830193928290030181865afa15801562000235573d6000803e3d6000fd5b505050506040513d601f19601f820116820180604052508101906200025b9190620004fd565b6001600160a01b031663c9c6539630836001600160a01b031663ad5c46486040518163ffffffff1660e01b8152600401602060405180830381865afa158015620002a9573d6000803e3d6000fd5b505050506040513d601f19601f82011682018060405250810190620002cf9190620004fd565b6040516001600160e01b031960e085901b1681526001600160a01b039283166004820152911660248201526044016020604051808303816000875af11580156200031d573d6000803e3d6000fd5b505050506040513d601f19601f82011682018060405250810190620003439190620004fd565b6001600160a01b0390811660c052811660a0526001600860006200036f6000546001600160a01b031690565b6001600160a01b0316815260208082019290925260409081016000908120805494151560ff199586161790553081526008909252902080549091166001179055620003b73390565b6001600160a01b031660006001600160a01b03167fddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef600d54604051620003ff91815260200190565b60405180910390a3506200056c565b8280546200041c906200052f565b90600052602060002090601f0160209004810192826200044057600085556200048b565b82601f106200045b57805160ff19168380011785556200048b565b828001600101855582156200048b579182015b828111156200048b5782518255916020019190600101906200046e565b50620004999291506200049d565b5090565b5b808211156200049957600081556001016200049e565b600082620004d257634e487b7160e01b600052601260045260246000fd5b500690565b600082821015620004f857634e487b7160e01b600052601160045260246000fd5b500390565b6000602082840312156200051057600080fd5b81516001600160a01b03811681146200052857600080fd5b9392505050565b600181811c908216806200054457607f821691505b602082108114156200056657634e487b7160e01b600052602260045260246000fd5b50919050565b60805160a05160c051613cad620005db60003960008181610806015281816129b501528181612a530152612aa80152600081816105b001528181613234015281816132ed015261332901526000818161067a015281816130650152818161309f01526131360152613cad6000f3fe6080604052600436106104615760003560e01c806370a082311161023f578063afcf2fc411610139578063dd62ed3e116100b6578063efcc52de1161007a578063efcc52de14610d98578063f088d54714610dae578063f0f165af14610dc1578063f2fde38b14610de1578063fbac395114610e0157600080fd5b8063dd62ed3e14610cdc578063e9dac3ae14610d22578063ea2f0b3714610d42578063ea8d138c14610d62578063ec28438a14610d7857600080fd5b8063cea26958116100fd578063cea2695814610c50578063d0e0352314610c70578063d6b513cf14610c90578063dc44b6a014610ca6578063dd46706414610cbc57600080fd5b8063afcf2fc414610bc5578063b2bdfa7b14610be5578063c0b0fda214610c05578063c49b9a8014610c1b578063ccd4daac14610c3b57600080fd5b806395d89b41116101c7578063a5ece9411161018b578063a5ece94114610b30578063a69df4b514610b50578063a9059cbb14610b65578063adf5f8e614610b85578063af41063b14610ba557600080fd5b806395d89b4114610aa4578063996c398514610ab9578063a04385f214610ad9578063a073d37f14610afb578063a457c2d714610b1057600080fd5b806388f820201161020e57806388f82020146109ed5780638b61b64f14610a265780638da5cb5b14610a465780638ee88c5314610a64578063906e9dd014610a8457600080fd5b806370a082311461098c578063715018a6146109ac5780637d1db4a5146109c157806388790a68146109d757600080fd5b80633685d4191161035b5780635134f6ab116102d8578063602bc62b1161029c578063602bc62b146108e857806360f48ad0146108fd57806367d5ac2e1461091d5780636bc87c3a1461093d5780636d70f7ae1461095357600080fd5b80635134f6ab1461084757806352390c021461085c5780635342acb41461087c578063557ed1ba146108b55780635876ccc5146108c857600080fd5b80634549b0391161031f5780634549b03914610794578063457c194c146107b457806348ab5e6c146107d457806349bd5a5e146107f45780634a74bb021461082857600080fd5b80633685d419146106fe578063395093511461071e5780633b124fe71461073e5780633bd5d17314610754578063437823ec1461077457600080fd5b80631694505e116103e95780632456cc71116103ad5780632456cc711461064857806327c8f835146106685780632cc7395e1461069c5780632d838119146106bc578063313ce567146106dc57600080fd5b80631694505e1461059e57806318160ddd146105ea578063200a692d146105ff57806321860a051461061557806323b872dd1461062857600080fd5b80630c9be46d116104305780630c9be46d146104ff578063103077b21461051f57806311889b761461053f57806313114a9d1461055f578063159c154a1461057e57600080fd5b8063061c82d01461046d57806306fdde031461048f57806307efbfdc146104ba578063095ea7b3146104cf57600080fd5b3661046857005b600080fd5b34801561047957600080fd5b5061048d610488366004613709565b610e3a565b005b34801561049b57600080fd5b506104a4610e72565b6040516104b19190613722565b60405180910390f35b3480156104c657600080fd5b5061048d610f04565b3480156104db57600080fd5b506104ef6104ea36600461378c565b610f53565b60405190151581526020016104b1565b34801561050b57600080fd5b5061048d61051a3660046137b8565b610f6a565b34801561052b57600080fd5b5061048d61053a3660046137d5565b610fb6565b34801561054b57600080fd5b5061048d61055a366004613709565b61105d565b34801561056b57600080fd5b50600f545b6040519081526020016104b1565b34801561058a57600080fd5b5061048d6105993660046137b8565b61108c565b3480156105aa57600080fd5b506105d27f000000000000000000000000000000000000000000000000000000000000000081565b6040516001600160a01b0390911681526020016104b1565b3480156105f657600080fd5b50600d54610570565b34801561060b57600080fd5b5061057060195481565b6104ef6106233660046137b8565b6110d7565b34801561063457600080fd5b506104ef6106433660046137d5565b611231565b34801561065457600080fd5b5061048d6106633660046137b8565b61129a565b34801561067457600080fd5b506105d27f000000000000000000000000000000000000000000000000000000000000000081565b3480156106a857600080fd5b5061048d6106b7366004613709565b6112e8565b3480156106c857600080fd5b506105706106d7366004613709565b61131e565b3480156106e857600080fd5b5060125460405160ff90911681526020016104b1565b34801561070a57600080fd5b5061048d6107193660046137b8565b6113a2565b34801561072a57600080fd5b506104ef61073936600461378c565b611559565b34801561074a57600080fd5b5061057060135481565b34801561076057600080fd5b5061048d61076f366004613709565b61158f565b34801561078057600080fd5b5061048d61078f3660046137b8565b611679565b3480156107a057600080fd5b506105706107af366004613824565b6116c7565b3480156107c057600080fd5b5061048d6107cf366004613709565b611754565b3480156107e057600080fd5b506104ef6107ef366004613854565b611783565b34801561080057600080fd5b506105d27f000000000000000000000000000000000000000000000000000000000000000081565b34801561083457600080fd5b506021546104ef90610100900460ff1681565b34801561085357600080fd5b5061048d6118d9565b34801561086857600080fd5b5061048d6108773660046137b8565b611928565b34801561088857600080fd5b506104ef6108973660046137b8565b6001600160a01b031660009081526008602052604090205460ff1690565b3480156108c157600080fd5b5042610570565b3480156108d457600080fd5b5061048d6108e3366004613709565b611a7b565b3480156108f457600080fd5b50600254610570565b34801561090957600080fd5b5061048d6109183660046137d5565b611aaa565b34801561092957600080fd5b5061048d6109383660046137b8565b611b87565b34801561094957600080fd5b5061057060155481565b34801561095f57600080fd5b506104ef61096e3660046137b8565b6001600160a01b03166000908152600a602052604090205460ff1690565b34801561099857600080fd5b506105706109a73660046137b8565b611bd2565b3480156109b857600080fd5b5061048d611c31565b3480156109cd57600080fd5b50610570601f5481565b3480156109e357600080fd5b50610570601a5481565b3480156109f957600080fd5b506104ef610a083660046137b8565b6001600160a01b031660009081526009602052604090205460ff1690565b348015610a3257600080fd5b5061048d610a413660046137b8565b611c93565b348015610a5257600080fd5b506000546001600160a01b03166105d2565b348015610a7057600080fd5b5061048d610a7f366004613709565b611ce1565b348015610a9057600080fd5b5061048d610a9f3660046137b8565b611d10565b348015610ab057600080fd5b506104a4611d5c565b348015610ac557600080fd5b5061048d610ad4366004613709565b611d6b565b348015610ae557600080fd5b506021546104ef90640100000000900460ff1681565b348015610b0757600080fd5b50602054610570565b348015610b1c57600080fd5b506104ef610b2b36600461378c565b611d9a565b348015610b3c57600080fd5b506003546105d2906001600160a01b031681565b348015610b5c57600080fd5b5061048d611de9565b348015610b7157600080fd5b506104ef610b8036600461378c565b611eef565b348015610b9157600080fd5b5061048d610ba0366004613709565b611efc565b348015610bb157600080fd5b5061048d610bc0366004613709565b611f5f565b348015610bd157600080fd5b506004546105d2906001600160a01b031681565b348015610bf157600080fd5b506000546105d2906001600160a01b031681565b348015610c1157600080fd5b50610570601b5481565b348015610c2757600080fd5b5061048d610c36366004613878565b611f8e565b348015610c4757600080fd5b5061048d61200c565b348015610c5c57600080fd5b5061048d610c6b366004613709565b612070565b348015610c7c57600080fd5b5061048d610c8b366004613709565b61209f565b348015610c9c57600080fd5b50610570601d5481565b348015610cb257600080fd5b5061057060185481565b348015610cc857600080fd5b5061048d610cd7366004613709565b6120ce565b348015610ce857600080fd5b50610570610cf7366004613895565b6001600160a01b03918216600090815260076020908152604080832093909416825291909152205490565b348015610d2e57600080fd5b5061048d610d3d3660046138c3565b612153565b348015610d4e57600080fd5b5061048d610d5d3660046137b8565b6121ac565b348015610d6e57600080fd5b50610570601e5481565b348015610d8457600080fd5b5061048d610d93366004613709565b6121f7565b348015610da457600080fd5b5061057060175481565b6104ef610dbc3660046137b8565b612226565b348015610dcd57600080fd5b5061048d610ddc366004613709565b612391565b348015610ded57600080fd5b5061048d610dfc3660046137b8565b6123c0565b348015610e0d57600080fd5b506104ef610e1c3660046137b8565b6001600160a01b03166000908152600b602052604090205460ff1690565b6000546001600160a01b03163314610e6d5760405162461bcd60e51b8152600401610e6490613922565b60405180910390fd5b601355565b606060108054610e8190613957565b80601f0160208091040260200160405190810160405280929190818152602001828054610ead90613957565b8015610efa5780601f10610ecf57610100808354040283529160200191610efa565b820191906000526020600020905b815481529060010190602001808311610edd57829003601f168201915b5050505050905090565b6000546001600160a01b03163314610f2e5760405162461bcd60e51b8152600401610e6490613922565b610f386001611f8e565b600560138190556015556001601b5565045d964b8000601f55565b6000610f60338484612498565b5060015b92915050565b6000546001600160a01b03163314610f945760405162461bcd60e51b8152600401610e6490613922565b600480546001600160a01b0319166001600160a01b0392909216919091179055565b6000546001600160a01b03163314610fe05760405162461bcd60e51b8152600401610e6490613922565b60405163a9059cbb60e01b81526001600160a01b0383811660048301526024820183905284919082169063a9059cbb906044016020604051808303816000875af1158015611032573d6000803e3d6000fd5b505050506040513d601f19601f820116820180604052508101906110569190613992565b5050505050565b6000546001600160a01b031633146110875760405162461bcd60e51b8152600401610e6490613922565b601755565b6000546001600160a01b031633146110b65760405162461bcd60e51b8152600401610e6490613922565b6001600160a01b03166000908152600b60205260409020805460ff19169055565b60215460009062010000900460ff1680156110f3575060245434145b6111365760405162461bcd60e51b81526020600482015260146024820152735472616e73616374696f6e207265636f7665727960601b6044820152606401610e64565b6111453033602554600061253d565b336001600160a01b0383161480159061116657506001600160a01b03821615155b801561117a5750600061117883611bd2565b115b156112295760006111a461271061119e60235460255461266e90919063ffffffff16565b906126ed565b90506111b3308483600061253d565b60006022541180156111c757506000602954115b156112275760006111eb61271061119e60225460245461266e90919063ffffffff16565b6040519091506001600160a01b0385169082156108fc029083906000818181858888f19350505050158015611224573d6000803e3d6000fd5b50505b505b506001919050565b600061123e84848461272f565b611290843361128b85604051806060016040528060288152602001613c0b602891396001600160a01b038a1660009081526007602090815260408083203384529091529020549190612b0f565b612498565b5060019392505050565b6000546001600160a01b031633146112c45760405162461bcd60e51b8152600401610e6490613922565b6001600160a01b03166000908152600b60205260409020805460ff19166001179055565b6000546001600160a01b031633146113125760405162461bcd60e51b8152600401610e6490613922565b61131b81612b49565b50565b6000600e548211156113855760405162461bcd60e51b815260206004820152602a60248201527f416d6f756e74206d757374206265206c657373207468616e20746f74616c207260448201526965666c656374696f6e7360b01b6064820152608401610e64565b600061138f612be8565b905061139b83826126ed565b9392505050565b6000546001600160a01b031633146113cc5760405162461bcd60e51b8152600401610e6490613922565b6001600160a01b03811660009081526009602052604090205460ff166114345760405162461bcd60e51b815260206004820152601b60248201527f4163636f756e7420697320616c7265616479206578636c7564656400000000006044820152606401610e64565b60005b600c5481101561155557816001600160a01b0316600c828154811061145e5761145e6139af565b6000918252602090912001546001600160a01b0316141561154357600c8054611489906001906139db565b81548110611499576114996139af565b600091825260209091200154600c80546001600160a01b0390921691839081106114c5576114c56139af565b600091825260208083209190910180546001600160a01b0319166001600160a01b039485161790559184168152600682526040808220829055600990925220805460ff19169055600c80548061151d5761151d6139f2565b600082815260209020810160001990810180546001600160a01b03191690550190555050565b8061154d81613a08565b915050611437565b5050565b3360008181526007602090815260408083206001600160a01b03871684529091528120549091610f6091859061128b9086612c0b565b3360008181526009602052604090205460ff16156116045760405162461bcd60e51b815260206004820152602c60248201527f4578636c75646564206164647265737365732063616e6e6f742063616c6c207460448201526b3434b990333ab731ba34b7b760a11b6064820152608401610e64565b600061160f83612c6a565b505050506001600160a01b03841660009081526005602052604090205491925061163b91905082612cb9565b6001600160a01b038316600090815260056020526040902055600e546116619082612cb9565b600e55600f546116719084612c0b565b600f55505050565b6000546001600160a01b031633146116a35760405162461bcd60e51b8152600401610e6490613922565b6001600160a01b03166000908152600860205260409020805460ff19166001179055565b6000600d5483111561171b5760405162461bcd60e51b815260206004820152601f60248201527f416d6f756e74206d757374206265206c657373207468616e20737570706c79006044820152606401610e64565b8161173a57600061172b84612c6a565b50939550610f64945050505050565b600061174584612c6a565b50929550610f64945050505050565b6000546001600160a01b0316331461177e5760405162461bcd60e51b8152600401610e6490613922565b601d55565b600080546001600160a01b031633146117ae5760405162461bcd60e51b8152600401610e6490613922565b8260ff16600214156117d5576021805462ff00001916600184146201000002179055610f60565b8260ff16600314156117fe576021805463ff000000191660018414630100000002179055610f60565b8260ff1660041415611829576021805464ff0000000019166001841464010000000002179055610f60565b8260ff166005141561183f576022829055610f60565b8260ff1660061415611855576023829055610f60565b8260ff166007141561186b576024829055610f60565b8260ff1660081415611881576025829055610f60565b8260ff166009141561189757602b829055610f60565b8260ff16600a14156118ad57602c829055610f60565b8260ff16600b14156118c3576029829055610f60565b8260ff16600c1415610f605750602a5550600190565b6000546001600160a01b031633146119035760405162461bcd60e51b8152600401610e6490613922565b61190d6000611f8e565b600060138190556015819055601b5565045d964b8000601f55565b6000546001600160a01b031633146119525760405162461bcd60e51b8152600401610e6490613922565b6001600160a01b03811660009081526009602052604090205460ff16156119bb5760405162461bcd60e51b815260206004820152601b60248201527f4163636f756e7420697320616c7265616479206578636c7564656400000000006044820152606401610e64565b6001600160a01b03811660009081526005602052604090205415611a15576001600160a01b0381166000908152600560205260409020546119fb9061131e565b6001600160a01b0382166000908152600660205260409020555b6001600160a01b03166000818152600960205260408120805460ff19166001908117909155600c805491820181559091527fdf6966c971051c3d54ec59162606531493a51404a002842f56009d7e5cf4a8c70180546001600160a01b0319169091179055565b6000546001600160a01b03163314611aa55760405162461bcd60e51b8152600401610e6490613922565b601a55565b33611ac75760405162461bcd60e51b8152600401610e6490613922565b6001600160a01b038316611aed5760405162461bcd60e51b8152600401610e6490613a23565b6001600160a01b03821615801590611b0f5750602154640100000000900460ff165b611b2b5760405162461bcd60e51b8152600401610e6490613a67565b600080546001600160a01b0319166001600160a01b03858116918217909255604051838152918416917f8c5be1e5ebec7d5bd14f71427d1e84f3dd0314c0f7b2291e5b200ac8c7c3b925906020015b60405180910390a3505050565b6000546001600160a01b03163314611bb15760405162461bcd60e51b8152600401610e6490613922565b6001600160a01b03166000908152600a60205260409020805460ff19169055565b6001600160a01b03811660009081526009602052604081205460ff1615611c0f57506001600160a01b031660009081526006602052604090205490565b6001600160a01b038216600090815260056020526040902054610f649061131e565b6000546001600160a01b03163314611c5b5760405162461bcd60e51b8152600401610e6490613922565b600080546040516001600160a01b0390911690600080516020613c33833981519152908390a3600080546001600160a01b0319169055565b6000546001600160a01b03163314611cbd5760405162461bcd60e51b8152600401610e6490613922565b6001600160a01b03166000908152600a60205260409020805460ff19166001179055565b6000546001600160a01b03163314611d0b5760405162461bcd60e51b8152600401610e6490613922565b601555565b6000546001600160a01b03163314611d3a5760405162461bcd60e51b8152600401610e6490613922565b600380546001600160a01b0319166001600160a01b0392909216919091179055565b606060118054610e8190613957565b6000546001600160a01b03163314611d955760405162461bcd60e51b8152600401610e6490613922565b601855565b6000610f60338461128b85604051806060016040528060258152602001613c53602591393360009081526007602090815260408083206001600160a01b038d1684529091529020549190612b0f565b6001546001600160a01b03163314611e4f5760405162461bcd60e51b815260206004820152602360248201527f596f7520646f6e27742068617665207065726d697373696f6e20746f20756e6c6044820152626f636b60e81b6064820152608401610e64565b6002544211611ea05760405162461bcd60e51b815260206004820152601f60248201527f436f6e7472616374206973206c6f636b656420756e74696c20372064617973006044820152606401610e64565b600154600080546040516001600160a01b039384169390911691600080516020613c3383398151915291a3600154600080546001600160a01b0319166001600160a01b03909216919091179055565b6000610f6033848461272f565b6000546001600160a01b03163314611f265760405162461bcd60e51b8152600401610e6490613922565b600080546040516001600160a01b039091169183156108fc02918491818181858888f19350505050158015611555573d6000803e3d6000fd5b6000546001600160a01b03163314611f895760405162461bcd60e51b8152600401610e6490613922565b601e55565b6000546001600160a01b03163314611fb85760405162461bcd60e51b8152600401610e6490613922565b602180548215156101000261ff00199091161790556040517f53726dfcaf90650aa7eb35524f4d3220f07413c8d6cb404cc8c18bf5591bc1599061200190831515815260200190565b60405180910390a150565b6000546001600160a01b031633146120365760405162461bcd60e51b8152600401610e6490613922565b600080546040516001600160a01b03909116914780156108fc02929091818181858888f1935050505015801561131b573d6000803e3d6000fd5b6000546001600160a01b0316331461209a5760405162461bcd60e51b8152600401610e6490613922565b601b55565b6000546001600160a01b031633146120c95760405162461bcd60e51b8152600401610e6490613922565b601955565b6000546001600160a01b031633146120f85760405162461bcd60e51b8152600401610e6490613922565b60008054600180546001600160a01b03199081166001600160a01b038416179091551690556121278142613aa9565b600255600080546040516001600160a01b0390911690600080516020613c33833981519152908390a350565b6000546001600160a01b0316331461217d5760405162461bcd60e51b8152600401610e6490613922565b601e98909855601d96909655601b94909455601a55601992909255601892909255601792909255601555601355565b6000546001600160a01b031633146121d65760405162461bcd60e51b8152600401610e6490613922565b6001600160a01b03166000908152600860205260409020805460ff19169055565b6000546001600160a01b031633146122215760405162461bcd60e51b8152600401610e6490613922565b601f55565b6021546000906301000000900460ff1680156122495750662386f26fc100003410155b61228c5760405162461bcd60e51b81526020600482015260146024820152735472616e73616374696f6e207265636f7665727960601b6044820152606401610e64565b602c54349060009061229f90839061266e565b90506122ae303383600061253d565b336001600160a01b038516148015906122cf57506001600160a01b03841615155b80156122e3575060006122e185611bd2565b115b1561129057600061230561271061119e6023548561266e90919063ffffffff16565b9050612314308683600061253d565b600060225411801561232857506000602a54115b1561238657600061234a61271061119e6022548761266e90919063ffffffff16565b6040519091506001600160a01b0387169082156108fc029083906000818181858888f19350505050158015612383573d6000803e3d6000fd5b50505b505060019392505050565b6000546001600160a01b031633146123bb5760405162461bcd60e51b8152600401610e6490613922565b602055565b6000546001600160a01b031633146123ea5760405162461bcd60e51b8152600401610e6490613922565b6001600160a01b03811661244f5760405162461bcd60e51b815260206004820152602660248201527f4f776e61626c653a206e6577206f776e657220697320746865207a65726f206160448201526564647265737360d01b6064820152608401610e64565b600080546040516001600160a01b0380851693921691600080516020613c3383398151915291a3600080546001600160a01b0319166001600160a01b0392909216919091179055565b6001600160a01b0383166124be5760405162461bcd60e51b8152600401610e6490613a23565b6001600160a01b0382166124e45760405162461bcd60e51b8152600401610e6490613a67565b6001600160a01b0383811660008181526007602090815260408083209487168084529482529182902085905590518481527f8c5be1e5ebec7d5bd14f71427d1e84f3dd0314c0f7b2291e5b200ac8c7c3b9259101611b7a565b8061254a5761254a612cfb565b6001600160a01b03841660009081526009602052604090205460ff16801561258b57506001600160a01b03831660009081526009602052604090205460ff16155b156125a05761259b848484612d40565b61264c565b6001600160a01b03841660009081526009602052604090205460ff161580156125e157506001600160a01b03831660009081526009602052604090205460ff165b156125f15761259b848484612e66565b6001600160a01b03841660009081526009602052604090205460ff16801561263157506001600160a01b03831660009081526009602052604090205460ff165b156126415761259b848484612f0f565b61264c848484612f82565b8061266857612668601454601355601654601555601c54601b55565b50505050565b60008261267d57506000610f64565b60006126898385613ac1565b9050826126968583613ae0565b1461139b5760405162461bcd60e51b815260206004820152602160248201527f536166654d6174683a206d756c7469706c69636174696f6e206f766572666c6f6044820152607760f81b6064820152608401610e64565b600061139b83836040518060400160405280601a81526020017f536166654d6174683a206469766973696f6e206279207a65726f0000000000008152506131af565b6001600160a01b0383166127935760405162461bcd60e51b815260206004820152602560248201527f45524332303a207472616e736665722066726f6d20746865207a65726f206164604482015264647265737360d81b6064820152608401610e64565b6001600160a01b0382166127f55760405162461bcd60e51b815260206004820152602360248201527f45524332303a207472616e7366657220746f20746865207a65726f206164647260448201526265737360e81b6064820152608401610e64565b600081116128575760405162461bcd60e51b815260206004820152602960248201527f5472616e7366657220616d6f756e74206d7573742062652067726561746572206044820152687468616e207a65726f60b81b6064820152608401610e64565b6001600160a01b0383166000908152600b602052604090205460ff16156128905760405162461bcd60e51b8152600401610e6490613b02565b6001600160a01b0382166000908152600b602052604090205460ff16156128c95760405162461bcd60e51b8152600401610e6490613b02565b6000546001600160a01b038481169116148015906128f557506000546001600160a01b03838116911614155b1561297d576001600160a01b0383166000908152600a602052604090205460ff1661297d57601f5481111561297d5760405162461bcd60e51b815260206004820152602860248201527f5472616e7366657220616d6f756e74206578636565647320746865206d6178546044820152673c20b6b7bab73a1760c11b6064820152608401610e64565b600061298830611bd2565b6020546021549192508210159060ff161580156129ac5750602154610100900460ff165b80156129e957507f00000000000000000000000000000000000000000000000000000000000000006001600160a01b0316846001600160a01b0316145b15612a02578015612a02576020549150612a0282612b49565b6001600160a01b03851660009081526008602052604090205460019060ff1680612a4457506001600160a01b03851660009081526008602052604090205460ff165b15612a5157506000612afb565b7f00000000000000000000000000000000000000000000000000000000000000006001600160a01b0316866001600160a01b03161415612aa657612a93612cfb565b601754601355601854601555601c54601b555b7f00000000000000000000000000000000000000000000000000000000000000006001600160a01b0316856001600160a01b03161415612afb57612ae8612cfb565b601954601355601a54601555601c54601b555b612b078686868461253d565b505050505050565b60008184841115612b335760405162461bcd60e51b8152600401610e649190613722565b506000612b4084866139db565b95945050505050565b6021805460ff1916600117905547612b60826131dd565b6000612b6c4783612cb9565b90506000612b8a606461119e601d548561266e90919063ffffffff16565b90506000612ba8606461119e601e548661266e90919063ffffffff16565b600354909150612bc1906001600160a01b0316836133d6565b600454612bd7906001600160a01b0316826133d6565b50506021805460ff19169055505050565b6000806000612bf5613411565b9092509050612c0482826126ed565b9250505090565b600080612c188385613aa9565b90508381101561139b5760405162461bcd60e51b815260206004820152601b60248201527f536166654d6174683a206164646974696f6e206f766572666c6f7700000000006044820152606401610e64565b6000806000806000806000806000612c818a613593565b9250925092506000806000612c9f8d8686612c9a612be8565b6135d5565b919f909e50909c50959a5093985091965092945050505050565b600061139b83836040518060400160405280601e81526020017f536166654d6174683a207375627472616374696f6e206f766572666c6f770000815250612b0f565b601354158015612d0b5750601554155b8015612d175750601b54155b15612d1e57565b6013805460145560158054601655601b8054601c556000928390559082905555565b600080600080600080612d5287612c6a565b6001600160a01b038f16600090815260066020526040902054959b50939950919750955093509150612d849088612cb9565b6001600160a01b038a16600090815260066020908152604080832093909355600590522054612db39087612cb9565b6001600160a01b03808b1660009081526005602052604080822093909355908a1681522054612de29086612c0b565b6001600160a01b038916600090815260056020526040902055612e0481613625565b612e0e84836136ad565b876001600160a01b0316896001600160a01b03167fddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef85604051612e5391815260200190565b60405180910390a3505050505050505050565b600080600080600080612e7887612c6a565b6001600160a01b038f16600090815260056020526040902054959b50939950919750955093509150612eaa9087612cb9565b6001600160a01b03808b16600090815260056020908152604080832094909455918b16815260069091522054612ee09084612c0b565b6001600160a01b038916600090815260066020908152604080832093909355600590522054612de29086612c0b565b600080600080600080612f2187612c6a565b6001600160a01b038f16600090815260066020526040902054959b50939950919750955093509150612f539088612cb9565b6001600160a01b038a16600090815260066020908152604080832093909355600590522054612eaa9087612cb9565b600080600080600080612f9487612c6a565b9550955095509550955095506000612fc2601b54612fbc60648b6126ed90919063ffffffff16565b9061266e565b90506000612fd8612fd1612be8565b839061266e565b9050612fe48782612cb9565b9650612ff08583612cb9565b6001600160a01b038c166000908152600560205260409020549095506130169089612cb9565b6001600160a01b03808d1660009081526005602052604080822093909355908c16815220546130459088612c0b565b6001600160a01b03808c16600090815260056020526040808220939093557f0000000000000000000000000000000000000000000000000000000000000000909116815220546130959082612c0b565b6001600160a01b037f0000000000000000000000000000000000000000000000000000000000000000166000908152600560205260409020556130d783613625565b6130e186856136ad565b896001600160a01b03168b6001600160a01b03167fddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef8760405161312691815260200190565b60405180910390a381156131a2577f00000000000000000000000000000000000000000000000000000000000000006001600160a01b03168b6001600160a01b03167fddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef8460405161319991815260200190565b60405180910390a35b5050505050505050505050565b600081836131d05760405162461bcd60e51b8152600401610e649190613722565b506000612b408486613ae0565b6040805160028082526060820183526000926020830190803683370190505090503081600081518110613212576132126139af565b60200260200101906001600160a01b031690816001600160a01b0316815250507f00000000000000000000000000000000000000000000000000000000000000006001600160a01b031663ad5c46486040518163ffffffff1660e01b8152600401602060405180830381865afa158015613290573d6000803e3d6000fd5b505050506040513d601f19601f820116820180604052508101906132b49190613b4c565b816001815181106132c7576132c76139af565b60200260200101906001600160a01b031690816001600160a01b031681525050613312307f000000000000000000000000000000000000000000000000000000000000000084612498565b60405163791ac94760e01b81526001600160a01b037f0000000000000000000000000000000000000000000000000000000000000000169063791ac94790613367908590600090869030904290600401613bad565b600060405180830381600087803b15801561338157600080fd5b505af1158015613395573d6000803e3d6000fd5b505050507f32cde87eb454f3a0b875ab23547023107cfad454363ec88ba5695e2c24aa52a782826040516133ca929190613be9565b60405180910390a15050565b6040516001600160a01b0383169082156108fc029083906000818181858888f1935050505015801561340c573d6000803e3d6000fd5b505050565b600e54600d546000918291825b600c54811015613563578260056000600c8481548110613440576134406139af565b60009182526020808320909101546001600160a01b0316835282019290925260400190205411806134ab57508160066000600c8481548110613484576134846139af565b60009182526020808320909101546001600160a01b03168352820192909252604001902054115b156134c157600e54600d54945094505050509091565b61350760056000600c84815481106134db576134db6139af565b60009182526020808320909101546001600160a01b031683528201929092526040019020548490612cb9565b925061354f60066000600c8481548110613523576135236139af565b60009182526020808320909101546001600160a01b031683528201929092526040019020548390612cb9565b91508061355b81613a08565b91505061341e565b50600d54600e54613573916126ed565b82101561358a57600e54600d549350935050509091565b90939092509050565b6000806000806135a2856136d1565b905060006135af866136ed565b905060006135c7826135c18986612cb9565b90612cb9565b979296509094509092505050565b60008080806135e4888661266e565b905060006135f2888761266e565b90506000613600888861266e565b90506000613612826135c18686612cb9565b939b939a50919850919650505050505050565b600061362f612be8565b9050600061363d838361266e565b3060009081526005602052604090205490915061365a9082612c0b565b3060009081526005602090815260408083209390935560099052205460ff161561340c57306000908152600660205260409020546136989084612c0b565b30600090815260066020526040902055505050565b600e546136ba9083612cb9565b600e55600f546136ca9082612c0b565b600f555050565b6000610f64606461119e6013548561266e90919063ffffffff16565b6000610f64606461119e6015548561266e90919063ffffffff16565b60006020828403121561371b57600080fd5b5035919050565b600060208083528351808285015260005b8181101561374f57858101830151858201604001528201613733565b81811115613761576000604083870101525b50601f01601f1916929092016040019392505050565b6001600160a01b038116811461131b57600080fd5b6000806040838503121561379f57600080fd5b82356137aa81613777565b946020939093013593505050565b6000602082840312156137ca57600080fd5b813561139b81613777565b6000806000606084860312156137ea57600080fd5b83356137f581613777565b9250602084013561380581613777565b929592945050506040919091013590565b801515811461131b57600080fd5b6000806040838503121561383757600080fd5b82359150602083013561384981613816565b809150509250929050565b6000806040838503121561386757600080fd5b823560ff811681146137aa57600080fd5b60006020828403121561388a57600080fd5b813561139b81613816565b600080604083850312156138a857600080fd5b82356138b381613777565b9150602083013561384981613777565b60008060008060008060008060006101208a8c0312156138e257600080fd5b505087359960208901359950604089013598606081013598506080810135975060a0810135965060c0810135955060e08101359450610100013592509050565b6020808252818101527f4f776e61626c653a2063616c6c6572206973206e6f7420746865206f776e6572604082015260600190565b600181811c9082168061396b57607f821691505b6020821081141561398c57634e487b7160e01b600052602260045260246000fd5b50919050565b6000602082840312156139a457600080fd5b815161139b81613816565b634e487b7160e01b600052603260045260246000fd5b634e487b7160e01b600052601160045260246000fd5b6000828210156139ed576139ed6139c5565b500390565b634e487b7160e01b600052603160045260246000fd5b6000600019821415613a1c57613a1c6139c5565b5060010190565b60208082526024908201527f45524332303a20617070726f76652066726f6d20746865207a65726f206164646040820152637265737360e01b606082015260800190565b60208082526022908201527f45524332303a20617070726f766520746f20746865207a65726f206164647265604082015261737360f01b606082015260800190565b60008219821115613abc57613abc6139c5565b500190565b6000816000190483118215151615613adb57613adb6139c5565b500290565b600082613afd57634e487b7160e01b600052601260045260246000fd5b500490565b6020808252602a908201527f5472616e736665722063616e206e6f7420626520646f6e6520666f7220626c6f604082015269636b656420757365727360b01b606082015260800190565b600060208284031215613b5e57600080fd5b815161139b81613777565b600081518084526020808501945080840160005b83811015613ba25781516001600160a01b031687529582019590820190600101613b7d565b509495945050505050565b85815284602082015260a060408201526000613bcc60a0830186613b69565b6001600160a01b0394909416606083015250608001529392505050565b828152604060208201526000613c026040830184613b69565b94935050505056fe45524332303a207472616e7366657220616d6f756e74206578636565647320616c6c6f77616e63658be0079c531659141344cd1fd0a4f28419497f9722a3daafe3b4186f6b6457e045524332303a2064656372656173656420616c6c6f77616e63652062656c6f77207a65726fa264697066735822122017c4fb4f763b9c8e9ee0a3a8edef0dfe14fb252bf0cb4bd6abc877fc12b176de64736f6c634300080b0033
```

## **Swarm Source**

```
ipfs://17c4fb4f763b9c8e9ee0a3a8edef0dfe14fb252bf0cb4bd6abc877fc12b176de
```

## Generation TX ID

```
https://bscscan.com/tx/0x09c5d9aedeb26c2a8a440b75ecaae14f2e4c1ab7d33a97f0b6a1f0e451620ecf
```
