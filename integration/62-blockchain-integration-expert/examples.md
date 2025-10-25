# Blockchain Integration Examples

## Example: NFT Minting Platform

```javascript
const { ethers } = require('ethers');
const pinataSDK = require('@pinata/sdk');

const pinata = new pinataSDK(process.env.PINATA_KEY, process.env.PINATA_SECRET);

async function uploadToIPFS(metadata) {
  const result = await pinata.pinJSONToIPFS(metadata);
  return `ipfs://${result.IpfsHash}`;
}

async function mintNFT(to, metadata) {
  // Upload metadata to IPFS
  const metadataURI = await uploadToIPFS(metadata);

  // Mint NFT
  const { signer } = await connectWallet();
  const nft = new ethers.Contract(NFT_CONTRACT, NFT_ABI, signer);

  const tx = await nft.mint(to, metadataURI);
  const receipt = await tx.wait();

  return { tokenId: receipt.logs[0].args.tokenId, metadataURI };
}
```

## Example: Cryptocurrency Payment

```javascript
async function acceptPayment(amount) {
  const { signer, address } = await connectWallet();

  const tx = await signer.sendTransaction({
    to: MERCHANT_ADDRESS,
    value: ethers.parseEther(amount.toString()),
  });

  const receipt = await tx.wait();

  await db.payments.create({
    from: address,
    amount,
    txHash: receipt.hash,
    status: 'confirmed',
  });

  return receipt;
}
```
