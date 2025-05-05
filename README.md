# B25
Asset Issuance Platform with sBTC Integration
import React, { useState, useCallback } from 'react';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Textarea } from '@/components/ui/textarea';
import {
  Card,
  CardContent,
  CardDescription,
  CardFooter,
  CardHeader,
  CardTitle,
} from '@/components/ui/card';
import { Tabs, TabsContent, TabsList, TabsTrigger } from '@/components/ui/tabs';
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from '@/components/ui/select';
import { cn } from '@/lib/utils';
import {
    Circle,
    CheckCircle,
    PlusCircle,
    Coins,
    Gem,
    Loader2,
    AlertTriangle,
    Info
} from 'lucide-react';

// ===============================
// Constants
// ===============================

const TOKEN_TYPES = [
  { value: 'fungible', label: 'Fungible (SIP-010)' },
  { value: 'nft', label: 'Non-Fungible (SIP-009)' },
];

// ===============================
// Helper Functions (Simulated)
// ===============================

// Simulate deploying a contract (replace with actual deployment logic)
const deployContract = async (clarityCode: string): Promise<string> => {
    // Simulate an async operation
    await new Promise((resolve) => setTimeout(resolve, 2000));
    // In a real app, this would interact with a Stacks client
    // to deploy the contract and get the contract ID.
    const contractId = `contract-${Math.random().toString(36).substring(7)}`;
    console.log('Contract deployed:', contractId, 'Code:', clarityCode);
    return contractId;
};

// Simulate token creation (replace with actual Clarity call)
const createToken = async (
  contractId: string,
  tokenType: string,
  name: string,
  supply: number,
  metadata: string,
): Promise<boolean> => {
  await new Promise((resolve) => setTimeout(resolve, 1500));
  console.log(
    `Creating ${tokenType} token "${name}" with supply ${supply} and metadata "${metadata}" in contract ${contractId}`,
  );
  return true; // Simulate success
};

// Simulate adding a market (sBTC pair)
const addMarket = async (contractId: string): Promise<boolean> => {
    await new Promise((resolve) => setTimeout(resolve, 1000));
    console.log(`Adding sBTC market for contract: ${contractId}`);
    return true;
};

// ===============================
// Components
// ===============================

const TokenCreationForm = ({
  contractId,
  onTokenCreated,
}: {
  contractId: string;
  onTokenCreated: () => void;
}) => {
  const [tokenType, setTokenType] = useState(TOKEN_TYPES[0].value);
  const [name, setName] = useState('');
  const [supply, setSupply] = useState<number | string>(''); // Use union type
  const [metadata, setMetadata] = useState('');
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  const [isCreated, setIsCreated] = useState(false);

  const handleCreateToken = useCallback(async () => {
    if (!name.trim()) {
      setError('Token name is required.');
      return;
    }
    if (!supply) {
      setError('Token supply is required.');
      return;
    }

    const supplyNum = Number(supply);
    if (isNaN(supplyNum) || supplyNum <= 0) {
        setError('Token supply must be a positive number.');
        return;
    }

    setIsLoading(true);
    setError(null);
    try {
      const success = await createToken(
        contractId,
        tokenType,
        name,
        supplyNum,
        metadata,
      );
      if (success) {
        setIsCreated(true);
        onTokenCreated(); // Notify parent component
      } else {
        setError('Token creation failed.'); // Generic error
      }
    } catch (err: any) {
      setError(err.message || 'An error occurred.');
    } finally {
      setIsLoading(false);
    }
  }, [contractId, tokenType, name, supply, metadata, onTokenCreated]);

  return (
    <Card>
      <CardHeader>
        <CardTitle>Create Token</CardTitle>
        <CardDescription>
          Define the properties of your new token.
        </CardDescription>
      </CardHeader>
      <CardContent className="space-y-4">
        <Select onValueChange={setTokenType} value={tokenType}>
          <SelectTrigger>
            <SelectValue placeholder="Token Type" />
          </SelectTrigger>
          <SelectContent>
            {TOKEN_TYPES.map((type) => (
              <SelectItem key={type.value} value={type.value}>
                {type.label}
              </SelectItem>
            ))}
          </SelectContent>
        </Select>
        <Input
          placeholder="Token Name"
          value={name}
          onChange={(e) => setName(e.target.value)}
          disabled={isLoading}
        />
        <Input
          placeholder="Initial Supply"
          value={supply}
          onChange={(e) => {
              const value = e.target.value;
              // Allow only numbers and empty string
              if (/^(\d+(\.\d{0,})?)?$/.test(value)) {
                  setSupply(value);
              }
          }}
          disabled={isLoading}
        />
        <Textarea
          placeholder="Metadata (optional)"
          value={metadata}
          onChange={(e) => setMetadata(e.target.value)}
          disabled={isLoading}
        />
        {error && (
          <div className="text-red-500 text-sm flex items-center gap-1">
            <AlertTriangle className="w-4 h-4" />
            {error}
          </div>
        )}
        {isCreated && (
          <div className="text-green-500 text-sm flex items-center gap-1">
            <CheckCircle className="w-4 h-4" />
            Token created successfully!
          </div>
        )}
      </CardContent>
      <CardFooter>
        <Button
          onClick={handleCreateToken}
          disabled={isLoading || isCreated}
          className="w-full"
        >
          {isLoading ? (
            <>
              <Loader2 className="mr-2 h-4 w-4 animate-spin" />
              Creating...
            </>
          ) : (
            'Create Token'
          )}
        </Button>
      </CardFooter>
    </Card>
  );
};

const MarketSetupForm = ({ contractId }: { contractId: string }) => {
  const [isMarketCreated, setIsMarketCreated] = useState(false);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

    const handleAddMarket = useCallback(async () => {
        setIsLoading(true);
        setError(null);
        try {
            const success = await addMarket(contractId);
            if (success) {
                setIsMarketCreated(true);
            } else {
                setError('Failed to add market.');
            }
        } catch (err: any) {
            setError(err.message);
        } finally {
            setIsLoading(false);
        }
    }, [contractId]);

  return (
    <Card>
      <CardHeader>
        <CardTitle>Set up Market</CardTitle>
        <CardDescription>
          Enable trading for your token with sBTC.
        </CardDescription>
      </CardHeader>
      <CardContent>
        {isMarketCreated ? (
          <div className="text-green-500 text-sm flex items-center gap-1">
            <CheckCircle className="w-4 h-4" />
            Market for sBTC pair is active!
          </div>
        ) : (
          <p className="text-gray-500 text-sm">
            To allow trading your token with sBTC, you need to set up a market.
          </p>
        )}
        {error && (
          <div className="text-red-500 text-sm flex items-center gap-1 mt-4">
            <AlertTriangle className="w-4 h-4" />
            {error}
          </div>
        )}
      </CardContent>
      <CardFooter>
        <Button
          onClick={handleAddMarket}
          disabled={isMarketCreated || isLoading}
          className="w-full"
        >
          {isLoading ? (
            <>
              <Loader2 className="mr-2 h-4 w-4 animate-spin" />
              Setting up...
            </>
          ) : (
            'Set up Market'
          )}
        </Button>
      </CardFooter>
    </Card>
  );
};

const AssetIssuancePlatform = () => {
  const [clarityCode, setClarityCode] = useState('');
  const [contractId, setContractId] = useState<string | null>(null);
  const [isDeployed, setIsDeployed] = useState(false);
  const [deploymentLoading, setDeploymentLoading] = useState(false);
  const [deploymentError, setDeploymentError] = useState<string | null>(null);
  const [tokenCreated, setTokenCreated] = useState(false);

  const handleDeploy = useCallback(async () => {
    setDeploymentLoading(true);
    setDeploymentError(null);
    try {
      const newContractId = await deployContract(clarityCode);
      setContractId(newContractId);
      setIsDeployed(true);
    } catch (err: any) {
      setDeploymentError(err.message || 'Failed to deploy contract.');
      setIsDeployed(false);
    } finally {
      setDeploymentLoading(false);
    }
  }, [clarityCode]);

    const handleTokenCreated = () => {
        setTokenCreated(true);
    };

  return (
    <div className="p-6">
      <h1 className="text-3xl font-bold mb-6 flex items-center gap-2">
        <Coins className="w-8 h-8 text-yellow-500" />
        Asset Issuance Platform
      </h1>
      <p className="text-gray-500 mb-8">
        Create and manage your own tokens on the Stacks blockchain. Integrate with
        sBTC for trading and utility.
      </p>

      <Tabs defaultValue="deploy" className="w-full space-y-6">
        <TabsList className="grid w-full grid-cols-2">
          <TabsTrigger value="deploy">
            <Gem className='w-4 h-4 mr-2'/>
            Deploy Contract
          </TabsTrigger>
          <TabsTrigger value="manage">
          <PlusCircle className='w-4 h-4 mr-2'/>
            Manage Tokens
          </TabsTrigger>
        </TabsList>
        <TabsContent value="deploy" className="space-y-4">
          <Card>
            <CardHeader>
              <CardTitle>Deploy Smart Contract</CardTitle>
              <CardDescription>
                Enter your Clarity code to deploy the token contract.
              </CardDescription>
            </CardHeader>
            <CardContent>
              <Textarea
                placeholder="Clarity Code (Token Contract)"
                value={clarityCode}
                onChange={(e) => setClarityCode(e.target.value)}
                className="min-h-[200px]"
                disabled={deploymentLoading || isDeployed}
              />
              {deploymentError && (
                <div className="text-red-500 text-sm flex items-center gap-1 mt-2">
                  <AlertTriangle className="w-4 h-4" />
                  {deploymentError}
                </div>
              )}
            </CardContent>
            <CardFooter>
              <Button
                onClick={handleDeploy}
                disabled={deploymentLoading || isDeployed}
                className="w-full"
              >
                {deploymentLoading ? (
                  <>
                    <Loader2 className="mr-2 h-4 w-4 animate-spin" />
                    Deploying...
                  </>
                ) : (
                  'Deploy'
                )}
              </Button>
            </CardFooter>
          </Card>
          {isDeployed && (
            <div className="bg-green-100 border border-green-400 text-green-700 px-4 py-3 rounded relative" role="alert">
              <strong className="font-bold">Contract Deployed! </strong>
              <span className="block sm:inline">Contract ID: {contractId}</span>
            </div>
          )}
        </TabsContent>
        <TabsContent value="manage" className="space-y-4">
          {isDeployed ? (
            <>
              <TokenCreationForm contractId={contractId!} onTokenCreated={handleTokenCreated}/>
              {tokenCreated && <MarketSetupForm contractId={contractId!} />}
            </>
          ) : (
            <Card>
                <CardHeader>
                    <CardTitle>
                        <Info className='w-4 h-4 mr-2'/>
                        Before you begin
                    </CardTitle>
                    <CardDescription>
                        Deploy a contract to create and manage tokens.
                    </CardDescription>
                </CardHeader>
                <CardContent>
                    <p className='text-gray-500'>
                        Please deploy your smart contract first. After successful deployment, you can create tokens and set up a market for trading.
                    </p>
                </CardContent>
            </Card>
          )}
        </TabsContent>
      </Tabs>
    </div>
  );
};

export default AssetIssuancePlatform;
