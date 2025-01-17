# cairo-mmr

An implementation of Merkle Mountain Ranges in Cairo, using Pedersen hash. Should be used alongside an off-chain implementation, keeping track of all the node hashes, to generate the proofs and compute the peaks (unless you are using the stateless variant).

## Set Up

You should have [Protostar](https://docs.swmansion.com/protostar/) installed. See [installation](https://docs.swmansion.com/protostar/docs/tutorials/installation) docs.

### Project initialization

```bash
protostar init <your-project-name>
cd <your-project-name>
```

### Installing the library

```bash
protostar install HerodotusDev/cairo-mmr
```

## Usage

```cairo
// src/main.cairo

%lang starknet
from cairo_mmr.src.mmr import append, verify_proof
```

For demonstration purposes, in the next example we are going to keep track of the hashes and peaks on-chain. An off-chain solution should be used instead.

```cairo
func demo{syscall_ptr: felt*, range_check_ptr, pedersen_ptr: HashBuiltin*}() {
    alloc_locals;

    let (local peaks: felt*) = alloc();

    append(elem=1, peaks_len=0, peaks=peaks);

    let (node1) = hash2{hash_ptr=pedersen_ptr}(1, 1);
    assert peaks[0] = node1;

    append(elem=2, peaks_len=1, peaks=peaks);

    let (node2) = hash2{hash_ptr=pedersen_ptr}(2, 2);
    let (node3_1) = hash2{hash_ptr=pedersen_ptr}(node1, node2);
    let (node3) = hash2{hash_ptr=pedersen_ptr}(3, node3_1);

    let (local peaks: felt*) = alloc();
    assert peaks[0] = node3;

    let (local proof: felt*) = alloc();
    assert proof[0] = node2;
    verify_proof(index=1, value=1, proof_len=1, proof=proof, peaks_len=1, peaks=peaks);

    let (local proof: felt*) = alloc();
    assert proof[0] = node1;
    verify_proof(index=2, value=2, proof_len=1, proof=proof, peaks_len=1, peaks=peaks);

    return ();
}
```

## Historical MMR

Sometimes you might need to verify "immutable proofs", i.e. proofs that remains valid even when the top root hash of the tree gets updated.

For that purpose, you can use the historical_mmr tree, it tracks every root hash and allows you to generate/verify proofs against it (you can check `tests/historical/*` to see some examples).

```cairo
%lang starknet
from cairo_mmr.src.historical_mmr import append, verify_past_proof, get_tree_size_to_root
```

## Stateless MMR

A stateless MMR that does not write/read from contract storage and relies only on passed function parameters.

```cairo
%lang starknet
from cairo_mmr.src.stateless_mmr import append, verify_proof
```

## Development

### Project set up

```bash
git clone git@github.com:HerodotusDev/cairo-mmr.git
cd cairo-mmr
```

### Compile

```bash
protostar build
```

### Test

```bash
protostar test
```

## License

[GNU GPLv3](https://github.com/HerodotusDev/cairo-mmr/blob/main/LICENSE)
