# Swap
Erc20
import java.math.BigInteger;
import java.util.Arrays;
import org.web3j.abi.FunctionEncoder;
import org.web3j.abi.FunctionReturnDecoder;
import org.web3j.abi.TypeReference;
import org.web3j.abi.datatypes.Address;
import org.web3j.abi.datatypes.Function;
import org.web3j.abi.datatypes.Type;
import org.web3j.abi.datatypes.Uint;
import org.web3j.abi.datatypes.generated.Uint256;
import org.web3j.crypto.Credentials;
import org.web3j.protocol.Web3j;
import org.web3j.protocol.core.DefaultBlockParameterName;
import org.web3j.protocol.core.methods.request.Transaction;
import org.web3j.protocol.core.methods.response.EthGetTransactionCount;
import org.web3j.protocol.core.methods.response.EthSendTransaction;
import org.web3j.protocol.http.HttpService;
import org.web3j.tx.gas.DefaultGasProvider;
import org.web3j.utils.Numeric;

public class SwapEthForUsdt {
    public static void main(String[] args) throws Exception {
        // connect to the Ethereum network
        Web3j web3 = Web3j.build(new HttpService("https://mainnet.infura.io/v3/<YOUR_INFURA_PROJECT_ID>"));

        // load your Ethereum account credentials
        Credentials credentials = Credentials.create("<YOUR_PRIVATE_KEY>");

        // set the contract address and ABI for the ERC20 token
        String contractAddress = "<ERC20_CONTRACT_ADDRESS>";
        String contractABI = "<ERC20_CONTRACT_ABI>";

        // create a new function to get the balance of USDT tokens in your account
        Function balanceOfFunction = new Function(
                "balanceOf",
                Arrays.asList(new Address(credentials.getAddress())),
                Arrays.asList(new TypeReference<Uint256>() {})
        );
        String encodedFunction = FunctionEncoder.encode(balanceOfFunction);

        // call the balanceOf function on the ERC20 token contract
        String response = web3.ethCall(
                Transaction.createEthCallTransaction(
                        credentials.getAddress(),
                        contractAddress,
                        encodedFunction
                ),
                DefaultBlockParameterName.LATEST
        ).send().getValue();

        // decode the response from the balanceOf function
        BigInteger usdtBalance = FunctionReturnDecoder.decode(
                response,
                balanceOfFunction.getOutputParameters()
        ).get(0).getValue();

        // set the address of the Uniswap V2 router contract
        String routerAddress = "0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D";

        // set the address of the WETH token contract
        String wethAddress = "0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2";

        // set the amount of ETH you want to swap for USDT
        BigInteger ethAmount = new BigInteger("1"); // 1 ETH

        // calculate the amount of WETH needed for the swap
        Function getAmountsOutFunction = new Function(
                "getAmountsOut",
                Arrays.asList(
                        new Uint256(ethAmount),
                        new Address(wethAddress),
                        new Address(contractAddress)
                ),
                Arrays.asList(new TypeReference<Uint256>() {})
        );
        encodedFunction = FunctionEncoder.encode(getAmountsOutFunction);

        BigInteger[] amountsOut = FunctionReturnDecoder.decode(
                web3.ethCall(
                        Transaction.createEthCallTransaction(
                                credentials.getAddress(),
                                routerAddress,
                                encodedFunction
                        ),
                        DefaultBlockParameterName.LATEST
                ).
