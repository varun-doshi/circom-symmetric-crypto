<div>
    <div>
        <img src="https://raw.githubusercontent.com/reclaimprotocol/.github/main/assets/banners/Circom.png"  />
    </div>
</div>

This library contains circom zero-knowledge proof circuits for symmetric crypto operations. The goal is to enable a user to prove that they have the key to a symmetric encrypted message without revealing the key.

The following algorithms are supported:
- `chacha20`
- `aes-256-ctr`
- `aes-128-ctr`
	- which includes any CTR implementation. For eg. aes-256-gcm
	- note: this is a WIP, and may be insecure (borrowed implementation from [electron labs](https://github.com/Electron-Labs/aes-circom))

It uses the `groth16` implementaion in `snarkjs` to generate the proof.

## Installation

```bash
npm install git+https://github.com/reclaimprotocol/circom-symmetric-crypto
```

If using on the browser, or nodejs, you will need to install `snarkjs` as well.

```bash
npm install snarkjs
```

## Usage

### Generating Proof

```ts
import { generateProof, verifyProof, makeLocalSnarkJsZkOperator } from '@reclaimprotocol/circom-symmetric-crypto'
import { createCipheriv, randomBytes } from 'crypto'

async function main() {
	const key = randomBytes(32)
	const iv = randomBytes(12)
	const algorithm = 'chacha20'
	const data = 'Hello World!'

	const cipher = createCipheriv('chacha20-poly1305', key, iv)
	const ciphertext = Buffer.concat([
		cipher.update(data),
		cipher.final()
	])

	// the operator is the abstract interface for
	// the snarkjs library to generate & verify the proof
	const operator = await makeLocalSnarkJsZkOperator(algorithm)
	// generate the proof that you have the key to the ciphertext
	const {
		// groth16-snarkjs proof as a JSON string
		proofJson,
		// the plaintext, obtained from the output of the circuit
		plaintext,
	} = await generateProof({
		algorithm,
		// key, iv & counter are the private inputs to the circuit
		privateInput: {
			key,
			iv,
			// this is the counter from which to start
			// the stream cipher. Read about
			// the counter here: https://en.wikipedia.org/wiki/Stream_cipher
			offset: 0
		},
		// the public ciphertext input to the circuit
		publicInput: { ciphertext },
		operator,
	})

	// you can check that the plaintext obtained from the circuit
	// is the same as the plaintext obtained from the ciphertext
	const plaintextBuffer = plaintext
		// slice in case the plaintext was padded
		.slice(0, data.length)
	// "Hello World!"
	console.log(Buffer.from(plaintextBuffer).toString())

	// you can verify the proof with the public inputs
	// and the proof JSON string
	await verifyProof({
		proof: {
			proofJson,
			plaintext,
			algorithm
		},
		// the public inputs to the circuit
		publicInput: { ciphertext },
		operator
	})
	console.log('Proof verified')
}

main()
```

### Verifying Proof

Continuing from the above example:

```ts
// will assert the proof is valid,
// otherwise it will throw an error
await verifyProof(
	{ proofJson, plaintext, algorithm: 'chacha20' },
	{ ciphertext },
	zkOperator
)
console.log('proof verified')

```

## Development

1. Clone the repository
2. Install dependencies via: `npm i`
3. Install [circom](https://docs.circom.io/getting-started/installation/)

### Running Tests

Run the tests via `npm run test`

## Building the Circuit

### Prerequisites
curl, jq

Official Ptau file for bn128 with 256k max constraints can be downloaded by running
```bash
npm run download:ptau
```

Build the circuits via `ALG={alg} npm run build:circuit`.
For eg. `ALG=chacha20 npm run build:circuit`
Note: `ALG` is the same as mentioned in the first section of this readme.

### Regenerating the Verification Key

1. Generate bls12-381 parameters via `npm run generate:ptau`
2. Fix `build-circuit.sh` to use `-p bls12381` parameter
   - note: we currently use BN-128 for our circuit, but plan to switch to BLs for greater security
   - zkey and ptau file verification is disabled right now due to a bug in the latest snarkJS version 0.7.0
3. TODO

## Contributing to Our Project

We're excited that you're interested in contributing to our project! Before you get started, please take a moment to review the following guidelines.

## Code of Conduct

Please read and follow our [Code of Conduct](https://github.com/reclaimprotocol/.github/blob/main/Code-of-Conduct.md) to ensure a positive and inclusive environment for all contributors.

## Security

If you discover any security-related issues, please refer to our [Security Policy](https://github.com/reclaimprotocol/.github/blob/main/SECURITY.md) for information on how to responsibly disclose vulnerabilities.

## Contributor License Agreement

Before contributing to this project, please read and sign our [Contributor License Agreement (CLA)](https://github.com/reclaimprotocol/.github/blob/main/CLA.md).

## Indie Hackers

For Indie Hackers: [Check out our guidelines and potential grant opportunities](https://github.com/reclaimprotocol/.github/blob/main/Indie-Hackers.md)

## License

This project is licensed under a [custom license](https://github.com/reclaimprotocol/.github/blob/main/LICENSE). By contributing to this project, you agree that your contributions will be licensed under its terms.

Thank you for your contributions!
